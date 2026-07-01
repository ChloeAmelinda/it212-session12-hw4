# Unit Tests: Đăng ký tài khoản

File này bao gồm các class test cho `AccountServiceImpl` và `AccountController` sử dụng JUnit 5 và Mockito.

## 1. Unit Test cho Service: `AccountServiceImplTest.java`

```java
package com.bank.account.service.impl;

import com.bank.account.dto.AccountRequestDTO;
import com.bank.account.dto.AccountResponseDTO;
import com.bank.account.entity.Account;
import com.bank.account.repository.AccountRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class AccountServiceImplTest {

    @Mock
    private AccountRepository accountRepository;

    @InjectMocks
    private AccountServiceImpl accountService;

    private AccountRequestDTO request;

    @BeforeEach
    void setUp() {
        request = new AccountRequestDTO();
        request.setFullName("Nguyễn Văn A");
        request.setPhone("0987654321");
        request.setEmail("nguyenvana@gmail.com");
        request.setCitizenId("030012345678");
    }

    @Test
    void createAccount_Success() {
        // Arrange: Giả lập CCCD chưa tồn tại
        when(accountRepository.existsByCitizenId(request.getCitizenId())).thenReturn(false);
        
        Account mockSavedAccount = new Account();
        mockSavedAccount.setAccountId(1L);
        mockSavedAccount.setAccountNumber("1234567890");
        mockSavedAccount.setStatus("PENDING");
        
        // Giả lập lưu thành công
        when(accountRepository.save(any(Account.class))).thenReturn(mockSavedAccount);

        // Act: Gọi service
        AccountResponseDTO response = accountService.createAccount(request);

        // Assert: Kiểm tra kết quả
        assertNotNull(response);
        assertEquals(1L, response.getAccountId());
        assertEquals("1234567890", response.getAccountNumber());
        assertEquals("PENDING", response.getStatus());
        verify(accountRepository, times(1)).save(any(Account.class));
    }

    @Test
    void createAccount_Fail_DuplicateCitizenId() {
        // Arrange: Giả lập CCCD ĐÃ tồn tại
        when(accountRepository.existsByCitizenId(request.getCitizenId())).thenReturn(true);

        // Act & Assert: Bắt lỗi quăng ra IllegalArgumentException
        Exception exception = assertThrows(IllegalArgumentException.class, () -> {
            accountService.createAccount(request);
        });

        assertEquals("CCCD đã được sử dụng để đăng ký tài khoản", exception.getMessage());
        // Chắc chắn không có save() nào được gọi
        verify(accountRepository, never()).save(any(Account.class));
    }
}
```

## 2. Unit Test cho Controller: `AccountControllerTest.java`

```java
package com.bank.account.controller;

import com.bank.account.dto.AccountRequestDTO;
import com.bank.account.dto.AccountResponseDTO;
import com.bank.account.service.AccountService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(AccountController.class)
class AccountControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private AccountService accountService;

    @Autowired
    private ObjectMapper objectMapper;

    private AccountRequestDTO request;

    @BeforeEach
    void setUp() {
        request = new AccountRequestDTO();
        request.setFullName("Nguyễn Văn A");
        request.setPhone("0987654321");
        request.setEmail("nguyenvana@gmail.com");
        request.setCitizenId("030012345678");
    }

    @Test
    void registerAccount_Success() throws Exception {
        AccountResponseDTO response = AccountResponseDTO.builder()
                .accountId(1L)
                .accountNumber("1234567890")
                .status("PENDING")
                .build();

        when(accountService.createAccount(any(AccountRequestDTO.class))).thenReturn(response);

        mockMvc.perform(post("/api/v1/accounts/register")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.accountId").value(1))
                .andExpect(jsonPath("$.accountNumber").value("1234567890"))
                .andExpect(jsonPath("$.status").value("PENDING"));
    }
    
    @Test
    void registerAccount_Fail_Validation_MissingEmail() throws Exception {
        request.setEmail("invalid-email"); // Lỗi định dạng email không có @
        
        mockMvc.perform(post("/api/v1/accounts/register")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isBadRequest()); // Sẽ bị @Valid của Controller chặn lại và trả mã 400
    }
}
```
