# STM32 Secure Bootloader with XMODEM and AES-CMAC

A secure bootloader for the **STM32F411RE** microcontroller that ensures firmware integrity and authenticity using **AES-CMAC** cryptography. Firmware updates are transmitted over **UART using the XMODEM protocol**, enabling reliable and authenticated over-the-air or cable-based firmware upgrades.

---

---

## Features

**Secure Firmware Updates**
Firmware is transmitted over UART using the XMODEM protocol. The bootloader receives each packet, writes it to Flash memory, and verifies the complete image before execution.

**AES-CMAC Integrity Verification**
Each firmware binary is signed using AES-CMAC before transmission. Upon reception, the bootloader recomputes the CMAC over the received binary and compares it against the appended tag. Any mismatch blocks execution.

**Authenticity Enforcement**
Only firmware signed with the embedded key is accepted. If authentication fails, the bootloader outputs an error message over UART and halts — preventing execution of unauthorized or corrupted firmware.

**Simple UART Command Interface**

| Command | Action |
|---------|--------|
| `U` | Start firmware update via XMODEM |
| `J` | Verify and boot existing firmware |

---

## Bootloader Flow

```
[Bootloader Start]
       |
       v
[Send prompt over UART]
       |
       +---> User sends "U" ---> [Receive firmware via XMODEM]
       |                                  |
       |                         [Write to Flash]
       |                                  |
       +---> User sends "J"      [Verify AES-CMAC tag]
                    |                     |
                    v              [Valid?]
           [Verify AES-CMAC]      /       \
                    |           YES        NO
                    v            |          |
              [Valid?]     [Jump to app] [Reject + Error]
              /      \
            YES       NO
             |         |
      [Execute app] [Error + Halt]
```

---

## Python Firmware Signing Tool

Before transmission, the firmware binary must be signed using the provided Python script located in the `Tool/` directory.

The script performs the following steps:

1. Reads the compiled `.bin` firmware file.
2. Computes the **AES-CMAC** over the binary using the shared key.
3. Appends the CMAC tag and firmware size metadata to the binary.
4. Outputs a signed file (`app_with_cmac.bin`) to the `Output/` directory.

**Usage:**

```bash
python Tool/sign_firmware.py Application/build/app.bin
# Output: Output/app_with_cmac.bin
```

The signed binary is then sent to the bootloader using any XMODEM-compatible serial terminal (e.g., Tera Term, minicom).

---

## Security Details

**AES-CMAC Key**

The bootloader and signing tool share a 128-bit symmetric key:

```c
const uint8_t FW_AES_KEY[16] = {
    0x12, 0x34, 0x56, 0x78,
    0x9A, 0xBC, 0xDE, 0xF0,
    0x11, 0x22, 0x33, 0x44,
    0x55, 0x66, 0x77, 0x88
};
```

> **Note:** In a production deployment, this key should be stored in a protected memory region (e.g., OTP or secure enclave) and never exposed in source code.

**Why AES-CMAC?**

AES-CMAC (Cipher-based Message Authentication Code) provides both integrity and authenticity guarantees. It is a well-established standard (NIST SP 800-38B / RFC 4493) and is supported natively by the STM32 cryptographic library, making it well-suited for constrained embedded environments.

---

## Documentation

Full project documentation including system design, security rationale, and implementation details is available in:

```
docs/CMAC_SecurBoot.pdf
```

You can view it directly on GitHub:

[View Project Report (PDF)](CMAC_SecurBoot.pdf)

---

## Demo

The screenshot below shows a successful secure boot firmware update over UART:

![Secure Boot Update](CMAC.PNG)

---

## Requirements

- STM32F411RE Nucleo board
- STM32CubeIDE (for building Bootloader and Application)
- Python 3.x (for firmware signing script)
- XMODEM-capable serial terminal (Tera Term, minicom, etc.)
- STM32 Cryptographic Library (included in `STM32_Cryptographic/`)

---

## Getting Started

1. Build the **Bootloader** project in STM32CubeIDE and flash it to the MCU.
2. Build the **Application** project and locate the output `.bin` file.
3. Run the signing script to generate `app_with_cmac.bin`.
4. Connect via UART (115200 baud, 8N1).
5. Send `U` in the terminal, then transfer `app_with_cmac.bin` using XMODEM.
6. Once transfer completes, the bootloader verifies the CMAC and boots the application.

---

## License

This project is for educational and demonstration purposes.
