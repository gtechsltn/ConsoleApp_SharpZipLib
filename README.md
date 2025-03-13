# Console App (C#) using SharpZipLib with large file size

When dealing with large files using SharpZipLib, it's best to handle them in chunks to avoid excessive memory usage. Below is a C# example demonstrating efficient compression and decompression of large files using SharpZipLib's GZip stream.

## Install SharpZipLib
Run this command to install SharpZipLib via NuGet:
```
dotnet add package SharpZipLib
```

## Efficiently Compress and Decompress Large Files
```
using System;
using System.IO;
using ICSharpCode.SharpZipLib.GZip;

class LargeFileCompressor
{
    private const int BufferSize = 8192; // 8 KB buffer for efficient processing

    static void Main()
    {
        string inputFile = "largefile.txt";
        string compressedFile = "largefile.gz";
        string decompressedFile = "largefile_decompressed.txt";

        CompressLargeFile(inputFile, compressedFile);
        DecompressLargeFile(compressedFile, decompressedFile);
    }

    static void CompressLargeFile(string inputFile, string outputFile)
    {
        using (FileStream inFile = File.OpenRead(inputFile))
        using (FileStream outFile = File.Create(outputFile))
        using (GZipOutputStream gzipStream = new GZipOutputStream(outFile))
        {
            byte[] buffer = new byte[BufferSize];
            int bytesRead;
            while ((bytesRead = inFile.Read(buffer, 0, BufferSize)) > 0)
            {
                gzipStream.Write(buffer, 0, bytesRead);
            }
        }
        Console.WriteLine($"File compressed successfully: {outputFile}");
    }

    static void DecompressLargeFile(string inputFile, string outputFile)
    {
        using (FileStream inFile = File.OpenRead(inputFile))
        using (FileStream outFile = File.Create(outputFile))
        using (GZipInputStream gzipStream = new GZipInputStream(inFile))
        {
            byte[] buffer = new byte[BufferSize];
            int bytesRead;
            while ((bytesRead = gzipStream.Read(buffer, 0, BufferSize)) > 0)
            {
                outFile.Write(buffer, 0, bytesRead);
            }
        }
        Console.WriteLine($"File decompressed successfully: {outputFile}");
    }
}
```

## Why This Works Well for Large Files?
+ ✅ Chunk-Based Processing: Reads and writes in small chunks (8KB) instead of loading the whole file into memory.
+ ✅ Stream-Based: Uses streams to process data instead of holding it in memory.
✅ Efficient: Suitable for handling multi-gigabyte files without running out of memory.

# How to compress multiple files into a ZIP archive using SharpZipLib

## Compress Multiple Files into a ZIP Archive

This example takes multiple files from a folder and compresses them into a ZIP file.
```
using System;
using System.IO;
using ICSharpCode.SharpZipLib.Zip;

class MultiFileZipper
{
    static void Main()
    {
        string sourceFolder = "FilesToZip"; // Folder containing files to compress
        string zipFilePath = "output.zip";  // Output ZIP file

        CompressFilesToZip(sourceFolder, zipFilePath);
    }

    static void CompressFilesToZip(string folderPath, string zipFilePath)
    {
        using (FileStream zipFileStream = File.Create(zipFilePath))
        using (ZipOutputStream zipStream = new ZipOutputStream(zipFileStream))
        {
            zipStream.SetLevel(9); // Compression level (0 = no compression, 9 = max)

            foreach (string filePath in Directory.GetFiles(folderPath))
            {
                FileInfo fileInfo = new FileInfo(filePath);
                Console.WriteLine($"Adding: {fileInfo.Name}");

                using (FileStream fileStream = File.OpenRead(filePath))
                {
                    ZipEntry entry = new ZipEntry(fileInfo.Name)
                    {
                        Size = fileInfo.Length
                    };
                    zipStream.PutNextEntry(entry);

                    byte[] buffer = new byte[4096]; // 4 KB buffer
                    int bytesRead;
                    while ((bytesRead = fileStream.Read(buffer, 0, buffer.Length)) > 0)
                    {
                        zipStream.Write(buffer, 0, bytesRead);
                    }

                    zipStream.CloseEntry();
                }
            }
        }
        Console.WriteLine($"ZIP file created: {zipFilePath}");
    }
}
```

## How This Works?
+ ✅ Loops through all files in a folder.
+ ✅ Uses ZipOutputStream to write each file into a ZIP archive.
+ ✅ Uses a buffer (4 KB) for efficient memory handling, making it suitable for large files.
+ ✅ Sets compression level (zipStream.SetLevel(9)) for best compression.

# How to extract a ZIP archive using SharpZipLib
This will extract all files from a ZIP archive into a specified folder

## Extract a ZIP Archive
This example extracts all files from archive.zip into the ExtractedFiles folder.
```
using System;
using System.IO;
using ICSharpCode.SharpZipLib.Zip;

class ZipExtractor
{
    static void Main()
    {
        string zipFilePath = "archive.zip";  // Input ZIP file
        string extractFolder = "ExtractedFiles"; // Output folder

        ExtractZip(zipFilePath, extractFolder);
    }

    static void ExtractZip(string zipFilePath, string extractFolder)
    {
        if (!Directory.Exists(extractFolder))
        {
            Directory.CreateDirectory(extractFolder);
        }

        using (FileStream zipFileStream = File.OpenRead(zipFilePath))
        using (ZipInputStream zipStream = new ZipInputStream(zipFileStream))
        {
            ZipEntry entry;
            while ((entry = zipStream.GetNextEntry()) != null)
            {
                string outputPath = Path.Combine(extractFolder, entry.Name);
                Console.WriteLine($"Extracting: {entry.Name}");

                using (FileStream outputFileStream = File.Create(outputPath))
                {
                    byte[] buffer = new byte[4096]; // 4 KB buffer
                    int bytesRead;
                    while ((bytesRead = zipStream.Read(buffer, 0, buffer.Length)) > 0)
                    {
                        outputFileStream.Write(buffer, 0, bytesRead);
                    }
                }
            }
        }
        Console.WriteLine($"ZIP extracted to: {extractFolder}");
    }
}
```

## How This Works?
+ ✅ Opens a ZIP file using ZipInputStream.
+ ✅ Iterates through all entries (files) inside the ZIP using GetNextEntry().
+ ✅ Extracts each file to the output folder while maintaining original names.
+ ✅ Uses a buffer (4 KB) for efficient memory handling, making it suitable for large ZIP files.

# Extract Only Specific Files from a ZIP Archive
This example extracts only .txt files from archive.zip into the ExtractedFiles folder.
```
using System;
using System.IO;
using ICSharpCode.SharpZipLib.Zip;

class SelectiveZipExtractor
{
    static void Main()
    {
        string zipFilePath = "archive.zip";  // Input ZIP file
        string extractFolder = "ExtractedFiles"; // Output folder
        string fileExtensionToExtract = ".txt"; // Extract only .txt files

        ExtractSpecificFiles(zipFilePath, extractFolder, fileExtensionToExtract);
    }

    static void ExtractSpecificFiles(string zipFilePath, string extractFolder, string extensionFilter)
    {
        if (!Directory.Exists(extractFolder))
        {
            Directory.CreateDirectory(extractFolder);
        }

        using (FileStream zipFileStream = File.OpenRead(zipFilePath))
        using (ZipInputStream zipStream = new ZipInputStream(zipFileStream))
        {
            ZipEntry entry;
            while ((entry = zipStream.GetNextEntry()) != null)
            {
                if (!entry.IsFile || !entry.Name.EndsWith(extensionFilter, StringComparison.OrdinalIgnoreCase))
                {
                    continue; // Skip directories and files that don't match the filter
                }

                string outputPath = Path.Combine(extractFolder, Path.GetFileName(entry.Name));
                Console.WriteLine($"Extracting: {entry.Name}");

                using (FileStream outputFileStream = File.Create(outputPath))
                {
                    byte[] buffer = new byte[4096]; // 4 KB buffer
                    int bytesRead;
                    while ((bytesRead = zipStream.Read(buffer, 0, buffer.Length)) > 0)
                    {
                        outputFileStream.Write(buffer, 0, bytesRead);
                    }
                }
            }
        }
        Console.WriteLine($"ZIP extracted to: {extractFolder}");
    }
}
```

## How This Works?
+ ✅ Filters files by extension (.txt in this example).
+ ✅ Skips directories inside the ZIP.
+ ✅ Extracts only matching files to avoid unnecessary file processing.
+ ✅ Uses a buffer (4 KB) for efficient memory usage, making it suitable for large ZIP files.

# Extract Password-Protected ZIP Files Using SharpZipLib

SharpZipLib supports password-protected ZIP files, but you need to set the password before extracting. Below is a C# example that extracts password-protected ZIP files.

```
using System;
using System.IO;
using ICSharpCode.SharpZipLib.Zip;

class PasswordProtectedZipExtractor
{
    static void Main()
    {
        string zipFilePath = "protected.zip";  // Password-protected ZIP file
        string extractFolder = "ExtractedFiles"; // Output folder
        string zipPassword = "yourpassword";  // Set the correct password here

        ExtractPasswordProtectedZip(zipFilePath, extractFolder, zipPassword);
    }

    static void ExtractPasswordProtectedZip(string zipFilePath, string extractFolder, string password)
    {
        if (!Directory.Exists(extractFolder))
        {
            Directory.CreateDirectory(extractFolder);
        }

        using (FileStream zipFileStream = File.OpenRead(zipFilePath))
        using (ZipInputStream zipStream = new ZipInputStream(zipFileStream))
        {
            zipStream.Password = password; // Set the password

            ZipEntry entry;
            while ((entry = zipStream.GetNextEntry()) != null)
            {
                if (!entry.IsFile) continue; // Skip directories

                string outputPath = Path.Combine(extractFolder, Path.GetFileName(entry.Name));
                Console.WriteLine($"Extracting: {entry.Name}");

                using (FileStream outputFileStream = File.Create(outputPath))
                {
                    byte[] buffer = new byte[4096]; // 4 KB buffer
                    int bytesRead;
                    while ((bytesRead = zipStream.Read(buffer, 0, buffer.Length)) > 0)
                    {
                        outputFileStream.Write(buffer, 0, bytesRead);
                    }
                }
            }
        }
        Console.WriteLine($"ZIP extracted to: {extractFolder}");
    }
}
```

## How This Works?
+ ✅ Sets a password using zipStream.Password = "yourpassword" before extracting.
+ ✅ Skips directories and extracts only files.
+ ✅ Uses a buffer (4 KB) for memory efficiency.
+ ✅ Handles large password-protected ZIP files without loading everything into memory.

# Extract Password-Protected ZIP Files with Exception Handling

This version of the code handles exceptions properly, including:

✔ Wrong password detection
✔ File not found errors
✔ Invalid ZIP file handling

C# Code: Extract Password-Protected ZIP with Error Handling

```
using System;
using System.IO;
using ICSharpCode.SharpZipLib.Zip;

class SecureZipExtractor
{
    static void Main()
    {
        string zipFilePath = "protected.zip";  // Password-protected ZIP file
        string extractFolder = "ExtractedFiles"; // Output folder
        string zipPassword = "yourpassword";  // Set the correct password here

        try
        {
            ExtractPasswordProtectedZip(zipFilePath, extractFolder, zipPassword);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error: {ex.Message}");
        }
    }

    static void ExtractPasswordProtectedZip(string zipFilePath, string extractFolder, string password)
    {
        // Validate if the file exists
        if (!File.Exists(zipFilePath))
        {
            throw new FileNotFoundException("ZIP file not found!", zipFilePath);
        }

        // Ensure the extraction folder exists
        if (!Directory.Exists(extractFolder))
        {
            Directory.CreateDirectory(extractFolder);
        }

        try
        {
            using (FileStream zipFileStream = File.OpenRead(zipFilePath))
            using (ZipInputStream zipStream = new ZipInputStream(zipFileStream))
            {
                zipStream.Password = password; // Set the password

                ZipEntry entry;
                while ((entry = zipStream.GetNextEntry()) != null)
                {
                    if (!entry.IsFile) continue; // Skip directories

                    string outputPath = Path.Combine(extractFolder, Path.GetFileName(entry.Name));

                    Console.WriteLine($"Extracting: {entry.Name}");

                    try
                    {
                        using (FileStream outputFileStream = File.Create(outputPath))
                        {
                            byte[] buffer = new byte[4096]; // 4 KB buffer
                            int bytesRead;
                            while ((bytesRead = zipStream.Read(buffer, 0, buffer.Length)) > 0)
                            {
                                outputFileStream.Write(buffer, 0, bytesRead);
                            }
                        }
                    }
                    catch (Exception fileEx)
                    {
                        Console.WriteLine($"Failed to extract {entry.Name}: {fileEx.Message}");
                    }
                }
            }

            Console.WriteLine("ZIP extraction completed.");
        }
        catch (ZipException zipEx)
        {
            Console.WriteLine("Error: Invalid ZIP file or incorrect password.");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Unexpected error: {ex.Message}");
        }
    }
}
```

## What's Improved?
+ ✅ Handles wrong password cases – SharpZipLib doesn't throw a direct error for wrong passwords, but invalid entries indicate it.
+ ✅ Handles missing files – If the ZIP file is not found, it throws a FileNotFoundException.
+ ✅ Catches file writing errors – If a file cannot be extracted due to permission issues, it logs the error.
+ ✅ Handles corrupt ZIP files – Catches ZipException if the ZIP file is corrupted.
+ ✅ Ensures extraction folder exists – Automatically creates the output folder if it doesn't exist.

# Log Error into error.log file
```
using System;
using System.IO;
using ICSharpCode.SharpZipLib.Zip;

class SecureZipExtractor
{
    private static readonly string logFilePath = "error.log";

    static void Main()
    {
        string zipFilePath = "protected.zip";  // Password-protected ZIP file
        string extractFolder = "ExtractedFiles"; // Output folder
        string zipPassword = "yourpassword";  // Set the correct password here

        try
        {
            ExtractPasswordProtectedZip(zipFilePath, extractFolder, zipPassword);
        }
        catch (Exception ex)
        {
            LogError($"Critical Error: {ex.Message}");
        }
    }

    static void ExtractPasswordProtectedZip(string zipFilePath, string extractFolder, string password)
    {
        if (!File.Exists(zipFilePath))
        {
            throw new FileNotFoundException("ZIP file not found!", zipFilePath);
        }

        if (!Directory.Exists(extractFolder))
        {
            Directory.CreateDirectory(extractFolder);
        }

        try
        {
            using (FileStream zipFileStream = File.OpenRead(zipFilePath))
            using (ZipInputStream zipStream = new ZipInputStream(zipFileStream))
            {
                zipStream.Password = password; // Set the password

                ZipEntry entry;
                while ((entry = zipStream.GetNextEntry()) != null)
                {
                    if (!entry.IsFile) continue; // Skip directories

                    string outputPath = Path.Combine(extractFolder, Path.GetFileName(entry.Name));

                    try
                    {
                        using (FileStream outputFileStream = File.Create(outputPath))
                        {
                            byte[] buffer = new byte[4096]; // 4 KB buffer
                            int bytesRead;
                            while ((bytesRead = zipStream.Read(buffer, 0, buffer.Length)) > 0)
                            {
                                outputFileStream.Write(buffer, 0, bytesRead);
                            }
                        }
                        Console.WriteLine($"Extracted: {entry.Name}");
                    }
                    catch (Exception fileEx)
                    {
                        LogError($"Failed to extract {entry.Name}: {fileEx.Message}");
                    }
                }
            }

            Console.WriteLine("ZIP extraction completed.");
        }
        catch (ZipException zipEx)
        {
            LogError("Invalid ZIP file or incorrect password.");
        }
        catch (Exception ex)
        {
            LogError($"Unexpected error: {ex.Message}");
        }
    }

    static void LogError(string message)
    {
        try
        {
            File.AppendAllText(logFilePath, $"{DateTime.Now}: {message}{Environment.NewLine}");
        }
        catch
        {
            Console.WriteLine("Failed to write to log file.");
        }
    }
}
```

# Handle Corrupt or Invalid ZIP Files with SharpZipLib

This updated version detects if a ZIP file is corrupt, invalid, or encrypted with the wrong password.

```
using System;
using System.IO;
using ICSharpCode.SharpZipLib.Zip;

class SecureZipExtractor
{
    private static readonly string logFilePath = "error.log";

    static void Main()
    {
        string zipFilePath = "protected.zip";  // ZIP file path
        string extractFolder = "ExtractedFiles"; // Output folder
        string zipPassword = "yourpassword";  // Set the correct password here

        try
        {
            if (!IsValidZipFile(zipFilePath))
            {
                Console.WriteLine("Invalid or corrupted ZIP file.");
                LogError("Invalid or corrupted ZIP file.");
                return;
            }

            ExtractPasswordProtectedZip(zipFilePath, extractFolder, zipPassword);
        }
        catch (Exception ex)
        {
            LogError($"Critical Error: {ex.Message}");
        }
    }

    static bool IsValidZipFile(string zipFilePath)
    {
        try
        {
            using (FileStream zipFileStream = File.OpenRead(zipFilePath))
            using (ZipFile zip = new ZipFile(zipFileStream))
            {
                return zip.TestArchive(true); // Check if the ZIP archive is valid
            }
        }
        catch
        {
            return false; // If an exception occurs, the ZIP file is invalid or corrupted
        }
    }

    static void ExtractPasswordProtectedZip(string zipFilePath, string extractFolder, string password)
    {
        if (!File.Exists(zipFilePath))
        {
            throw new FileNotFoundException("ZIP file not found!", zipFilePath);
        }

        if (!Directory.Exists(extractFolder))
        {
            Directory.CreateDirectory(extractFolder);
        }

        try
        {
            using (FileStream zipFileStream = File.OpenRead(zipFilePath))
            using (ZipInputStream zipStream = new ZipInputStream(zipFileStream))
            {
                zipStream.Password = password; // Set the password

                ZipEntry entry;
                while ((entry = zipStream.GetNextEntry()) != null)
                {
                    if (!entry.IsFile) continue; // Skip directories

                    string outputPath = Path.Combine(extractFolder, Path.GetFileName(entry.Name));

                    try
                    {
                        using (FileStream outputFileStream = File.Create(outputPath))
                        {
                            byte[] buffer = new byte[4096]; // 4 KB buffer
                            int bytesRead;
                            while ((bytesRead = zipStream.Read(buffer, 0, buffer.Length)) > 0)
                            {
                                outputFileStream.Write(buffer, 0, bytesRead);
                            }
                        }
                        Console.WriteLine($"Extracted: {entry.Name}");
                    }
                    catch (Exception fileEx)
                    {
                        LogError($"Failed to extract {entry.Name}: {fileEx.Message}");
                    }
                }
            }

            Console.WriteLine("ZIP extraction completed.");
        }
        catch (ZipException zipEx)
        {
            LogError("Invalid ZIP file or incorrect password.");
        }
        catch (Exception ex)
        {
            LogError($"Unexpected error: {ex.Message}");
        }
    }

    static void LogError(string message)
    {
        try
        {
            File.AppendAllText(logFilePath, $"{DateTime.Now}: {message}{Environment.NewLine}");
        }
        catch
        {
            Console.WriteLine("Failed to write to log file.");
        }
    }
}
```

## How It Works?
+ ✅ Detects if a ZIP file is corrupt or invalid before extraction using TestArchive(true).
+ ✅ Logs errors to error.log instead of printing to the console.
+ ✅ Prevents extracting from corrupt ZIP files, avoiding unnecessary processing.
+ ✅ Handles incorrect passwords by catching ZipException.
+ ✅ Handles missing ZIP files by throwing a FileNotFoundException.

# Detect Encrypted ZIPs & Handle Missing Passwords
```
using System;
using System.IO;
using ICSharpCode.SharpZipLib.Zip;

class SecureZipExtractor
{
    private static readonly string logFilePath = "error.log";

    static void Main()
    {
        string zipFilePath = "protected.zip";  // ZIP file path
        string extractFolder = "ExtractedFiles"; // Output folder
        string zipPassword = "";  // Leave empty to test password detection

        try
        {
            if (!IsValidZipFile(zipFilePath))
            {
                Console.WriteLine("Invalid or corrupted ZIP file.");
                LogError("Invalid or corrupted ZIP file.");
                return;
            }

            if (IsZipEncrypted(zipFilePath) && string.IsNullOrEmpty(zipPassword))
            {
                Console.WriteLine("ZIP file is encrypted, but no password was provided.");
                LogError("ZIP file is encrypted, but no password was provided.");
                return;
            }

            ExtractPasswordProtectedZip(zipFilePath, extractFolder, zipPassword);
        }
        catch (Exception ex)
        {
            LogError($"Critical Error: {ex.Message}");
        }
    }

    static bool IsValidZipFile(string zipFilePath)
    {
        try
        {
            using (FileStream zipFileStream = File.OpenRead(zipFilePath))
            using (ZipFile zip = new ZipFile(zipFileStream))
            {
                return zip.TestArchive(true); // Check if the ZIP archive is valid
            }
        }
        catch
        {
            return false; // If an exception occurs, the ZIP file is invalid or corrupted
        }
    }

    static bool IsZipEncrypted(string zipFilePath)
    {
        try
        {
            using (FileStream zipFileStream = File.OpenRead(zipFilePath))
            using (ZipFile zip = new ZipFile(zipFileStream))
            {
                foreach (ZipEntry entry in zip)
                {
                    if (entry.IsCrypted) // Check if any file inside the ZIP is encrypted
                    {
                        return true;
                    }
                }
            }
        }
        catch (Exception ex)
        {
            LogError($"Error checking encryption: {ex.Message}");
        }
        return false;
    }

    static void ExtractPasswordProtectedZip(string zipFilePath, string extractFolder, string password)
    {
        if (!File.Exists(zipFilePath))
        {
            throw new FileNotFoundException("ZIP file not found!", zipFilePath);
        }

        if (!Directory.Exists(extractFolder))
        {
            Directory.CreateDirectory(extractFolder);
        }

        try
        {
            using (FileStream zipFileStream = File.OpenRead(zipFilePath))
            using (ZipInputStream zipStream = new ZipInputStream(zipFileStream))
            {
                if (!string.IsNullOrEmpty(password))
                {
                    zipStream.Password = password; // Set the password
                }

                ZipEntry entry;
                while ((entry = zipStream.GetNextEntry()) != null)
                {
                    if (!entry.IsFile) continue; // Skip directories

                    string outputPath = Path.Combine(extractFolder, Path.GetFileName(entry.Name));

                    try
                    {
                        using (FileStream outputFileStream = File.Create(outputPath))
                        {
                            byte[] buffer = new byte[4096]; // 4 KB buffer
                            int bytesRead;
                            while ((bytesRead = zipStream.Read(buffer, 0, buffer.Length)) > 0)
                            {
                                outputFileStream.Write(buffer, 0, bytesRead);
                            }
                        }
                        Console.WriteLine($"Extracted: {entry.Name}");
                    }
                    catch (Exception fileEx)
                    {
                        LogError($"Failed to extract {entry.Name}: {fileEx.Message}");
                    }
                }
            }

            Console.WriteLine("ZIP extraction completed.");
        }
        catch (ZipException zipEx)
        {
            LogError("Invalid ZIP file or incorrect password.");
        }
        catch (Exception ex)
        {
            LogError($"Unexpected error: {ex.Message}");
        }
    }

    static void LogError(string message)
    {
        try
        {
            File.AppendAllText(logFilePath, $"{DateTime.Now}: {message}{Environment.NewLine}");
        }
        catch
        {
            Console.WriteLine("Failed to write to log file.");
        }
    }
}
```

# Retry ZIP Extraction with User Input
```
using System;
using System.IO;
using ICSharpCode.SharpZipLib.Zip;

class SecureZipExtractor
{
    private static readonly string logFilePath = "error.log";

    static void Main()
    {
        string zipFilePath = "protected.zip";  // ZIP file path
        string extractFolder = "ExtractedFiles"; // Output folder
        string zipPassword = "";  // Leave empty initially

        try
        {
            if (!IsValidZipFile(zipFilePath))
            {
                Console.WriteLine("Invalid or corrupted ZIP file.");
                LogError("Invalid or corrupted ZIP file.");
                return;
            }

            if (IsZipEncrypted(zipFilePath))
            {
                do
                {
                    Console.Write("Enter password for ZIP file: ");
                    zipPassword = Console.ReadLine();

                    if (string.IsNullOrEmpty(zipPassword))
                    {
                        Console.WriteLine("Password cannot be empty.");
                        continue;
                    }

                } while (!TryExtractZip(zipFilePath, extractFolder, zipPassword));

                Console.WriteLine("ZIP extraction completed successfully.");
            }
            else
            {
                ExtractPasswordProtectedZip(zipFilePath, extractFolder, zipPassword);
            }
        }
        catch (Exception ex)
        {
            LogError($"Critical Error: {ex.Message}");
        }
    }

    static bool IsValidZipFile(string zipFilePath)
    {
        try
        {
            using (FileStream zipFileStream = File.OpenRead(zipFilePath))
            using (ZipFile zip = new ZipFile(zipFileStream))
            {
                return zip.TestArchive(true); // Check if the ZIP archive is valid
            }
        }
        catch
        {
            return false; // If an exception occurs, the ZIP file is invalid or corrupted
        }
    }

    static bool IsZipEncrypted(string zipFilePath)
    {
        try
        {
            using (FileStream zipFileStream = File.OpenRead(zipFilePath))
            using (ZipFile zip = new ZipFile(zipFileStream))
            {
                foreach (ZipEntry entry in zip)
                {
                    if (entry.IsCrypted) // Check if any file inside the ZIP is encrypted
                    {
                        return true;
                    }
                }
            }
        }
        catch (Exception ex)
        {
            LogError($"Error checking encryption: {ex.Message}");
        }
        return false;
    }

    static bool TryExtractZip(string zipFilePath, string extractFolder, string password)
    {
        try
        {
            ExtractPasswordProtectedZip(zipFilePath, extractFolder, password);
            return true; // Success
        }
        catch (ZipException)
        {
            Console.WriteLine("Incorrect password. Try again.");
            return false; // Retry
        }
        catch (Exception ex)
        {
            LogError($"Extraction failed: {ex.Message}");
            return false;
        }
    }

    static void ExtractPasswordProtectedZip(string zipFilePath, string extractFolder, string password)
    {
        if (!File.Exists(zipFilePath))
        {
            throw new FileNotFoundException("ZIP file not found!", zipFilePath);
        }

        if (!Directory.Exists(extractFolder))
        {
            Directory.CreateDirectory(extractFolder);
        }

        using (FileStream zipFileStream = File.OpenRead(zipFilePath))
        using (ZipInputStream zipStream = new ZipInputStream(zipFileStream))
        {
            zipStream.Password = password; // Set the password

            ZipEntry entry;
            while ((entry = zipStream.GetNextEntry()) != null)
            {
                if (!entry.IsFile) continue; // Skip directories

                string outputPath = Path.Combine(extractFolder, Path.GetFileName(entry.Name));

                using (FileStream outputFileStream = File.Create(outputPath))
                {
                    byte[] buffer = new byte[4096]; // 4 KB buffer
                    int bytesRead;
                    while ((bytesRead = zipStream.Read(buffer, 0, buffer.Length)) > 0)
                    {
                        outputFileStream.Write(buffer, 0, bytesRead);
                    }
                }
                Console.WriteLine($"Extracted: {entry.Name}");
            }
        }
    }

    static void LogError(string message)
    {
        try
        {
            File.AppendAllText(logFilePath, $"{DateTime.Now}: {message}{Environment.NewLine}");
        }
        catch
        {
            Console.WriteLine("Failed to write to log file.");
        }
    }
}
```

## How This Works?
+ ✅ Detects if a ZIP file is encrypted and prompts for a password if needed.
+ ✅ Allows retrying with a new password if extraction fails due to an incorrect password.
+ ✅ Logs errors to error.log instead of cluttering the console.
+ ✅ Prevents extraction if the ZIP is corrupt or missing.
