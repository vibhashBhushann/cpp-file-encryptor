# C++ File Encryptor

A command-line tool written in modern C++ to recursively find and encrypt or decrypt all files within a specified directory. This project serves as a demonstration of core software engineering principles including object-oriented design, modern memory management, build automation with Make, and a design for concurrent processing using multiprocessing.

## Key Features

- **Recursive File Processing**: Traverses a directory and all its subdirectories to find and process every file.
- **Modern C++ (C++17)**: Leverages modern features like the `<filesystem>` library for portable path manipulation, and smart pointers (`std::unique_ptr`) for robust memory management.
- **Object-Oriented Design**: Follows the Separation of Concerns principle with distinct classes for file I/O (`IO`), task management (`ProcessManagement`), and the core encryption logic (`Cryption`).
- **Build Automation**: Includes a Makefile for efficient compilation, linking, and cleanup of the project.
- **Concurrency with Multiprocessing**: Designed to execute encryption tasks in parallel by forking child processes, providing stability and process isolation.

## How to Build and Run

### Prerequisites

- A C++ compiler that supports C++17 (e.g., `g++` 8 or later)
- `make`

### 1. Setup

Before running, you must create a `.env` file in the root of the project to store the encryption key.

```bash
# Create the file
touch .env

# Add a number to the file to be used as the encryption key
echo "5" > .env
```

### 2. Build the Project

Use the provided Makefile to compile the two executables (`encrypt_decrypt` and `cryption`).

```bash
make
```

### 3. Run the Main Program

Execute the main program. It will prompt you to enter the directory path and the action you want to perform.

```bash
./encrypt_decrypt
```

You will be asked for input:

```
Enter the directory path: ./test
Enter the action (encrypt/decrypt): encrypt
```

The program will then process all files in the `./test` directory.

### 4. Clean Up

To remove all compiled object files and executables, run:

```bash
make clean
```

## Technical Deep Dive

### 1. Concurrency Model: Multiprocessing (fork/execv)

To achieve parallelism and speed up the processing of many files, this project is designed to use a multiprocessing model. While the current implementation in `ProcessManagement.cpp` is sequential for debugging purposes, the commented-out code demonstrates the intended concurrent architecture.

**How it Works:**

- **Fork**: The main `encrypt_decrypt` process iterates through the queue of tasks. For each task, it calls `fork()`, creating a new child process that is a near-identical copy of the parent.
- **Exec**: The child process then immediately calls `execv()` to replace its own process image with the lightweight `./cryption` executable. The file path and action are passed as a command-line argument.
- **Isolation**: Each file is encrypted in its own isolated process. A crash or error while processing one file will not affect the parent or any other sibling processes.
- **Wait**: The parent process waits for the child to complete before forking a new process for the next task. This prevents the creation of an overwhelming number of "zombie" processes.

This fork-exec model is a classic and robust UNIX pattern for parallelism, offering maximum stability at the cost of slightly higher process creation overhead compared to multithreading.

### 2. Memory Management: RAII and Smart Pointers

The project strictly adheres to the modern C++ idiom of RAII (Resource Acquisition Is Initialization) to prevent resource leaks.

- **`std::unique_ptr`**: Task objects are allocated on the heap and managed by `std::unique_ptr`. This smart pointer enforces exclusive ownership, guaranteeing that the memory for a Task is automatically deallocated when it is no longer needed (e.g., when popped from the queue). Ownership is efficiently transferred using `std::move`.
- **File Handles**: The `IO` class encapsulates the `std::fstream` object. The file is opened in the constructor and automatically closed in the destructor when the `IO` object goes out of scope, ensuring no file handles are left dangling.

### 3. In-Place File I/O

The encryption and decryption algorithm operates directly on the files in-place to minimize memory usage.

- The file is opened in both read and write binary mode (`std::ios::in | std::ios::out | std::ios::binary`).
- The code iterates through the file byte-by-byte using a get-seek-put pattern:
  - `get(char)`: Reads a byte and advances the read cursor.
  - `seekp(-1, std::ios::cur)`: Moves the write cursor back one position.
  - `put(char)`: Overwrites the original byte with the transformed byte.

This approach is highly memory-efficient as the entire file is never loaded into RAM.

## Future Improvements

- **Implement Full Concurrency**: Uncomment the fork-exec logic to enable parallel processing, or implement a thread pool for a potentially more performant multithreaded approach.
- **Strong Encryption**: Replace the demonstrative Caesar cipher with a robust, industry-standard symmetric encryption algorithm like AES-256.
- **Secure Key Management**: The key should not be stored in a plain text `.env` file. A future version could integrate with a secure vault or prompt the user for a password to derive the key.

## Project Structure

```
.
├── encrypt_decrypt     # Main executable
├── cryption            # Worker executable for encryption/decryption
├── Makefile            # Build automation
├── .env                # Encryption key (create this file)
└── src/                # Source files
    ├── IO.cpp
    ├── ProcessManagement.cpp
    └── Cryption.cpp
```

## Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the issues page.

## Disclaimer

⚠️ **Warning**: This project uses a simple Caesar cipher for demonstration purposes only. **Do not use this for any real-world encryption needs.** For production use, implement a proper encryption algorithm like AES-256 with secure key management.
