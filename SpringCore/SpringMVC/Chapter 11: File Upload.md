# Spring MVC — Chapter 11: File Upload

We're at the last major practical topic in our Spring MVC module.

```text
Spring MVC
├── ✅ DispatcherServlet
├── ✅ HandlerMapping
├── ✅ HandlerAdapter
├── ✅ Controllers & Mapping
├── ✅ @PathVariable / @RequestParam
├── ✅ @RequestBody / @ResponseBody
├── ✅ Model / ViewResolver
├── ✅ Data Binding / Validation
├── ✅ Exception Handling
├── ✅ Interceptors
└── ⏭️ File Upload ← Current
```

We'll use:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need file upload?

Suppose your application allows employees to upload:

```text
resume.pdf
profile.jpg
offer-letter.pdf
```

An HTTP request doesn't send a file like a normal Java `String` parameter.

We need support for:

```text
Browser
   ↓
File
   ↓
HTTP Request
   ↓
Spring MVC
   ↓
Java
```

Spring MVC provides multipart request handling for this.

---

# 2. What is a Multipart Request?

A normal request might contain:

```text
name=Rahul
salary=60000
```

A file upload request typically uses:

```http
Content-Type: multipart/form-data
```

The request can contain multiple parts:

```text
Part 1 → name
Part 2 → salary
Part 3 → resume.pdf
```

That's why it's called **multipart**.

---

# 3. What is `MultipartFile`?

Spring provides:

```java
MultipartFile
```

to represent an uploaded file.

It gives you information such as:

```java
file.getOriginalFilename()
file.getContentType()
file.getSize()
file.getBytes()
file.getInputStream()
file.isEmpty()
```

Think of it as a Spring wrapper around the uploaded file.

---

# 4. Simple Upload Controller

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @PostMapping("/upload")
    public String upload(
            @RequestParam("file") MultipartFile file) {

        System.out.println(
                "File Name: " + file.getOriginalFilename());

        System.out.println(
                "Size: " + file.getSize());

        return "File uploaded";
    }
}
```

The client sends:

```text
POST /employees/upload
Content-Type: multipart/form-data
```

with a form field named:

```text
file
```

---

# 5. How does Spring process it?

Flow:

```text
Client
  ↓
multipart/form-data
  ↓
Tomcat
  ↓
DispatcherServlet
  ↓
Multipart Processing
  ↓
@RequestParam("file")
  ↓
MultipartFile
  ↓
Controller
```

Spring extracts the uploaded file and provides it as a `MultipartFile`.

---

# 6. Important: `@RequestParam` here

You may wonder:

> "We used `@RequestParam` for query parameters earlier. Why are we using it for a file?"

Because `@RequestParam("file")` is also used to bind a multipart form field to the controller parameter.

Example:

```java
@RequestParam("file") MultipartFile file
```

means:

> Find the multipart field named `file` and give me its uploaded file.

---

# 7. Saving the File

Suppose you want to save it to disk.

```java
@PostMapping("/upload")
public String upload(
        @RequestParam("file") MultipartFile file)
        throws IOException {

    Path path = Paths.get(
            "uploads/" + file.getOriginalFilename());

    Files.copy(
            file.getInputStream(),
            path,
            StandardCopyOption.REPLACE_EXISTING);

    return "Uploaded";
}
```

The basic flow is:

```text
MultipartFile
   ↓
InputStream
   ↓
File System
```

---

# 8. Why should we be careful with `getOriginalFilename()`?

This is an important security point.

Don't blindly trust:

```java
file.getOriginalFilename()
```

The filename comes from the client.

For production systems, you should generally:

* Validate the file type.
* Validate the size.
* Generate a safe server-side filename.
* Avoid allowing user input to determine arbitrary filesystem paths.
* Store files outside sensitive application directories where appropriate.

For example:

```java
String safeFileName = UUID.randomUUID() + ".pdf";
```

This is safer than directly using the uploaded filename as a path.

---

# 9. Validate File Size

Suppose your application accepts files up to 5 MB.

You can check:

```java
if (file.getSize() > 5 * 1024 * 1024) {
    throw new IllegalArgumentException(
        "File too large");
}
```

In real Spring Boot applications, multipart size limits can also be configured through application configuration.

---

# 10. Validate Empty File

Always consider:

```java
if (file.isEmpty()) {
    throw new IllegalArgumentException(
        "File is empty");
}
```

---

# 11. Validate Content Type

Example:

```java
String contentType = file.getContentType();

if (!"application/pdf".equals(contentType)) {
    throw new IllegalArgumentException(
        "Only PDF files are allowed");
}
```

But an important production point:

> **Don't rely only on the client-provided `Content-Type` header.**

For sensitive uploads, inspect the actual file format/content as well.

---

# 12. Multiple File Upload

Suppose the user uploads:

```text
resume.pdf
photo.jpg
certificate.pdf
```

Controller:

```java
@PostMapping("/upload-multiple")
public String uploadMultiple(
        @RequestParam("files")
        List<MultipartFile> files) {

    for (MultipartFile file : files) {

        System.out.println(
                file.getOriginalFilename());
    }

    return "Uploaded";
}
```

Spring binds the multiple multipart fields into a collection.

---

# 13. File + Form Data

You can also upload a file together with normal fields.

Request:

```text
name = Rahul
salary = 60000
resume = resume.pdf
```

Controller:

```java
@PostMapping("/employee")
public String saveEmployee(
        @RequestParam String name,
        @RequestParam Double salary,
        @RequestParam MultipartFile resume) {

    return "Success";
}
```

---

# 14. Using DTO + MultipartFile

For more structured requests, you can combine form data and file handling appropriately, but don't confuse this with JSON `@RequestBody`.

A multipart request is different from:

```http
Content-Type: application/json
```

You cannot simply expect a JSON body and a separate multipart file in the same way. For complex multipart APIs, Spring supports multipart parts and `@RequestPart`, which we'll mention below.

---

# 15. `@RequestPart`

Suppose the request contains:

```text
JSON part + file part
```

Example:

```text
employee → JSON
resume   → PDF
```

Then `@RequestPart` is useful.

```java
@PostMapping("/employee")
public String save(
        @RequestPart("employee")
        EmployeeRequest employee,

        @RequestPart("resume")
        MultipartFile resume) {

    return "Success";
}
```

This is a more advanced but useful distinction.

---

# 16. `@RequestParam` vs `@RequestPart`

### `@RequestParam`

Commonly used for:

```text
form fields
simple multipart values
file field binding
```

Example:

```java
@RequestParam("file")
MultipartFile file
```

### `@RequestPart`

Useful when handling a specific multipart part, especially when a part itself contains structured data like JSON.

```java
@RequestPart("employee")
EmployeeRequest employee
```

For your experience level, know the difference conceptually.

---

# 17. File Download

File handling isn't only upload.

You may also need to return a file.

For example:

```text
GET /employees/101/resume
```

The application reads the file and returns it as an HTTP response.

Spring can use types such as:

```java
ResponseEntity<Resource>
```

Conceptually:

```text
File
 ↓
Resource
 ↓
HTTP Response
 ↓
Browser
```

We'll see this pattern again when we work with REST APIs.

---

# 18. Where is file upload used?

Common enterprise use cases:

```text
Employee Management
  → Resume Upload

E-commerce
  → Product Images

Banking
  → Documents

Insurance
  → Claim Documents

HR
  → Certificates

Support Systems
  → Attachments
```

---

# 19. Interview Questions

### What is `MultipartFile`?

> `MultipartFile` is Spring's abstraction for an uploaded multipart file. It provides access to file metadata and content.

### What content type is normally used for file uploads?

```text
multipart/form-data
```

### How do you upload a file in Spring MVC?

> Accept a multipart request and bind the uploaded part to a `MultipartFile`, commonly using `@RequestParam`.

### How do you upload multiple files?

For example:

```java
@RequestParam("files")
List<MultipartFile> files
```

### When would you use `@RequestPart`?

> When you need to bind a specific multipart part, especially when a multipart request contains structured content such as JSON plus a file.

### What security problems can occur with file upload?

* Malicious files
* Oversized files
* Path traversal
* Unsafe filenames
* Unexpected file types
* Executable content

---

# 20. Best Practices

A production file-upload API should generally:

```text
Validate size
     ↓
Validate type/content
     ↓
Generate safe filename
     ↓
Store securely
     ↓
Return file identifier/path
```

Avoid:

```java
Files.copy(...,
    Paths.get(file.getOriginalFilename()));
```

without validating and sanitizing the filename.

Also, for larger production systems, files are often stored in dedicated object storage rather than directly in the application server's local filesystem.

---

# 21. Complete Upload Flow

```text
Client
   │
   │ multipart/form-data
   ▼
Tomcat
   ▼
DispatcherServlet
   ▼
Multipart Processing
   ▼
MultipartFile
   ▼
Controller
   ▼
Validation
   ▼
Service
   ▼
File Storage
   ▼
Database stores file metadata
```

A very common architecture is:

```text
Actual File
   ↓
Object Storage / File Storage

Database
   ↓
fileName
fileId
contentType
size
storageLocation
```

Rather than putting large files directly into normal relational database rows.

---

# 📍 Spring MVC Status

```text
Spring MVC
│
├── ✅ MVC fundamentals
├── ✅ DispatcherServlet
├── ✅ HandlerMapping
├── ✅ HandlerAdapter
├── ✅ Controllers
├── ✅ Request Mapping
├── ✅ PathVariable
├── ✅ RequestParam
├── ✅ RequestBody
├── ✅ ResponseBody
├── ✅ Model / ModelAndView
├── ✅ ViewResolver
├── ✅ Data Binding
├── ✅ Validation
├── ✅ Exception Handling
├── ✅ Interceptors
├── ✅ File Upload
│
└── ⏭️ Final MVC Revision
       ↓
     Spring REST
```

### Next

We'll do a **Spring MVC quick revision + interview round** to connect the entire request lifecycle:

```text
HTTP Request
 ↓
Tomcat
 ↓
DispatcherServlet
 ↓
HandlerMapping
 ↓
HandlerAdapter
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
Response
```

Then we'll move into **Spring REST**, where we'll focus on REST principles, HTTP methods/status codes, `ResponseEntity`, DTOs, JSON, validation, and designing production-style REST APIs.
