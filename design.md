Thiết kế RESTful API: Quản lý Users và Tasks
1. Tổng quan
   Hệ thống quản lý User (người dùng) và Task (công việc). Mỗi công việc thuộc về một người dùng thông qua logic khóa ngoại:
   `Task.userId` tham chiếu đến `User.id`
   Một `User` có thể có nhiều `Task`
   Một `Task` chỉ thuộc về một `User` tại một thời điểm
   API tuân thủ nguyên tắc RESTful:
   Dùng danh từ số nhiều cho resource: `/users`, `/tasks`
   Dùng HTTP Method để thể hiện hành động:
   `GET`: lấy dữ liệu
   `POST`: tạo mới hoặc thực hiện thao tác gắn/liên kết
   `PATCH`: cập nhật một phần dữ liệu
   `DELETE`: xóa dữ liệu
   Dữ liệu gửi lên server dùng JSON với `@RequestBody`
   Các ID trên URL dùng `@PathVariable`
   Dữ liệu lọc/tìm kiếm dùng `@RequestParam`
---
2. Resource chính
   2.1. User
   Ví dụ đối tượng `User`:
```json
{
  "id": 1,
  "name": "Nguyen Van A",
  "email": "vana@example.com",
  "role": "member"
}
```
Các trường gợi ý:
Field	Kiểu dữ liệu	Bắt buộc	Mô tả
`id`	Long	Không khi tạo	Mã định danh người dùng
`name`	String	Có	Tên người dùng
`email`	String	Có	Email người dùng, nên là duy nhất
`role`	String	Có	Vai trò, ví dụ: `admin`, `manager`, `member`
2.2. Task
Ví dụ đối tượng `Task`:
```json
{
  "id": 101,
  "title": "Hoàn thành báo cáo",
  "description": "Viết báo cáo tiến độ dự án",
  "status": "todo",
  "priority": "high",
  "userId": 1
}
```
Các trường gợi ý:
Field	Kiểu dữ liệu	Bắt buộc	Mô tả
`id`	Long	Không khi tạo	Mã định danh công việc
`title`	String	Có	Tiêu đề công việc
`description`	String	Không	Mô tả chi tiết công việc
`status`	String	Có	Trạng thái: `todo`, `in_progress`, `done`
`priority`	String	Có	Độ ưu tiên: `low`, `medium`, `high`
`userId`	Long	Không/Có tùy API	ID người dùng được giao việc
---
3. Quy tắc validation cơ bản
   3.1. Validation cho User
   Khi tạo mới user:
   `name` không được rỗng
   `email` không được rỗng
   `email` phải đúng định dạng email
   `email` không được trùng với user khác
   `role` chỉ nhận một trong các giá trị: `admin`, `manager`, `member`
   Ví dụ request không hợp lệ:
```json
{
  "name": "",
  "email": "abc",
  "role": "super_user"
}
```
Response gợi ý:
```json
{
  "message": "Validation failed",
  "errors": [
    "name must not be blank",
    "email must be valid",
    "role must be one of: admin, manager, member"
  ]
}
```
3.2. Validation cho Task
Khi tạo mới task:
`title` không được rỗng
`status` chỉ nhận: `todo`, `in_progress`, `done`
`priority` chỉ nhận: `low`, `medium`, `high`
Nếu truyền `userId`, user tương ứng phải tồn tại
Khi cập nhật trạng thái task:
`status` không được rỗng
`status` chỉ nhận: `todo`, `in_progress`, `done`
Khi gắn task cho user:
`taskId` phải tồn tại
`userId` phải tồn tại
---
4. Danh sách Endpoint
   4.1. Lấy toàn bộ danh sách công việc
   Endpoint
```http
GET /tasks
```
Mô tả
Lấy toàn bộ danh sách công việc trong hệ thống.
Response thành công
Status code: `200 OK`
```json
[
  {
    "id": 101,
    "title": "Hoàn thành báo cáo",
    "description": "Viết báo cáo tiến độ dự án",
    "status": "todo",
    "priority": "high",
    "userId": 1
  },
  {
    "id": 102,
    "title": "Kiểm thử API",
    "description": "Test các endpoint chính",
    "status": "in_progress",
    "priority": "medium",
    "userId": 2
  }
]
```
---
4.2. Lấy toàn bộ danh sách người dùng
Endpoint
```http
GET /users
```
Mô tả
Lấy toàn bộ danh sách người dùng trong hệ thống.
Response thành công
Status code: `200 OK`
```json
[
  {
    "id": 1,
    "name": "Nguyen Van A",
    "email": "vana@example.com",
    "role": "member"
  },
  {
    "id": 2,
    "name": "Tran Thi B",
    "email": "thib@example.com",
    "role": "manager"
  }
]
```
---
4.3. Tạo mới công việc
Endpoint
```http
POST /tasks
```
Mô tả
Tạo mới một công việc. Dữ liệu công việc được gửi qua JSON body bằng `@RequestBody`.
Nếu muốn tạo task và gắn luôn cho user, có thể truyền `userId` trong body. Server cần kiểm tra user có tồn tại hay không.
Request body
```json
{
  "title": "Hoàn thành báo cáo",
  "description": "Viết báo cáo tiến độ dự án",
  "status": "todo",
  "priority": "high",
  "userId": 1
}
```
Response thành công
Status code: `201 Created`
```json
{
  "id": 101,
  "title": "Hoàn thành báo cáo",
  "description": "Viết báo cáo tiến độ dự án",
  "status": "todo",
  "priority": "high",
  "userId": 1
}
```
Response lỗi gợi ý
Nếu `userId` không tồn tại:
Status code: `404 Not Found`
```json
{
  "message": "User not found with id: 1"
}
```
Nếu dữ liệu không hợp lệ:
Status code: `400 Bad Request`
```json
{
  "message": "Validation failed",
  "errors": [
    "title must not be blank",
    "priority must be one of: low, medium, high"
  ]
}
```
---
4.4. Tạo mới người dùng
Endpoint
```http
POST /users
```
Mô tả
Tạo mới một người dùng. Dữ liệu người dùng được gửi qua JSON body bằng `@RequestBody`.
Request body
```json
{
  "name": "Nguyen Van A",
  "email": "vana@example.com",
  "role": "member"
}
```
Response thành công
Status code: `201 Created`
```json
{
  "id": 1,
  "name": "Nguyen Van A",
  "email": "vana@example.com",
  "role": "member"
}
```
Response lỗi gợi ý
Nếu email bị trùng:
Status code: `409 Conflict`
```json
{
  "message": "Email already exists"
}
```
Nếu dữ liệu không hợp lệ:
Status code: `400 Bad Request`
```json
{
  "message": "Validation failed",
  "errors": [
    "name must not be blank",
    "email must be valid"
  ]
}
```
---
4.5. Cập nhật trạng thái một công việc
Endpoint
```http
PATCH /tasks/{taskId}/status
```
Mô tả
Cập nhật một phần thông tin của task, cụ thể là trường `status`.
Dùng `PATCH` vì chỉ cập nhật một phần resource, không thay thế toàn bộ task.
Path variable
Tên	Kiểu dữ liệu	Mô tả
`taskId`	Long	ID của công việc cần cập nhật
Request body
```json
{
  "status": "done"
}
```
Response thành công
Status code: `200 OK`
```json
{
  "id": 101,
  "title": "Hoàn thành báo cáo",
  "description": "Viết báo cáo tiến độ dự án",
  "status": "done",
  "priority": "high",
  "userId": 1
}
```
Response lỗi gợi ý
Nếu task không tồn tại:
Status code: `404 Not Found`
```json
{
  "message": "Task not found with id: 101"
}
```
Nếu status không hợp lệ:
Status code: `400 Bad Request`
```json
{
  "message": "Validation failed",
  "errors": [
    "status must be one of: todo, in_progress, done"
  ]
}
```
---
4.6. Cập nhật vai trò của người dùng
Endpoint
```http
PATCH /users/{userId}/role
```
Mô tả
Cập nhật một phần thông tin của user, cụ thể là trường `role`.
Path variable
Tên	Kiểu dữ liệu	Mô tả
`userId`	Long	ID của người dùng cần cập nhật vai trò
Request body
```json
{
  "role": "manager"
}
```
Response thành công
Status code: `200 OK`
```json
{
  "id": 1,
  "name": "Nguyen Van A",
  "email": "vana@example.com",
  "role": "manager"
}
```
Response lỗi gợi ý
Nếu user không tồn tại:
Status code: `404 Not Found`
```json
{
  "message": "User not found with id: 1"
}
```
Nếu role không hợp lệ:
Status code: `400 Bad Request`
```json
{
  "message": "Validation failed",
  "errors": [
    "role must be one of: admin, manager, member"
  ]
}
```
---
4.7. Xóa một công việc
Endpoint
```http
DELETE /tasks/{taskId}
```
Mô tả
Xóa một công việc khỏi hệ thống.
Path variable
Tên	Kiểu dữ liệu	Mô tả
`taskId`	Long	ID của công việc cần xóa
Response thành công
Status code: `204 No Content`
Không có response body.
Response lỗi gợi ý
Nếu task không tồn tại:
Status code: `404 Not Found`
```json
{
  "message": "Task not found with id: 101"
}
```
---
4.8. Xóa một người dùng khỏi hệ thống
Endpoint
```http
DELETE /users/{userId}
```
Mô tả
Xóa một người dùng khỏi hệ thống.
Cần quy định rõ cách xử lý các task đang thuộc về user đó. Có 2 hướng phổ biến:
Không cho xóa nếu user còn task.
Cho xóa user và chuyển `userId` của các task liên quan thành `null`.
Trong thiết kế này, chọn hướng an toàn: không cho xóa user nếu user vẫn còn task.
Path variable
Tên	Kiểu dữ liệu	Mô tả
`userId`	Long	ID của người dùng cần xóa
Response thành công
Status code: `204 No Content`
Không có response body.
Response lỗi gợi ý
Nếu user không tồn tại:
Status code: `404 Not Found`
```json
{
  "message": "User not found with id: 1"
}
```
Nếu user vẫn còn task:
Status code: `409 Conflict`
```json
{
  "message": "Cannot delete user because this user still has assigned tasks"
}
```
---
4.9. Tìm các công việc có mức độ ưu tiên là `high`
Endpoint
```http
GET /tasks?priority=high
```
Mô tả
Tìm danh sách công việc có độ ưu tiên là `high`.
Dùng query parameter vì đây là thao tác lọc danh sách task.
Query parameter
Tên	Kiểu dữ liệu	Bắt buộc	Mô tả
`priority`	String	Có	Độ ưu tiên cần lọc, ví dụ: `high`
Response thành công
Status code: `200 OK`
```json
[
  {
    "id": 101,
    "title": "Hoàn thành báo cáo",
    "description": "Viết báo cáo tiến độ dự án",
    "status": "todo",
    "priority": "high",
    "userId": 1
  },
  {
    "id": 103,
    "title": "Sửa lỗi đăng nhập",
    "description": "Fix bug login trên mobile",
    "status": "in_progress",
    "priority": "high",
    "userId": 2
  }
]
```
---
4.10. Tìm các công việc có độ ưu tiên `high` và được giao cho user có id là `1`
Endpoint
```http
GET /tasks?priority=high&userId=1
```
Mô tả
Tìm danh sách công việc thỏa mãn đồng thời 2 điều kiện:
`priority = high`
`userId = 1`
Query parameter
Tên	Kiểu dữ liệu	Bắt buộc	Mô tả
`priority`	String	Có	Độ ưu tiên cần lọc
`userId`	Long	Có	ID người dùng được giao công việc
Response thành công
Status code: `200 OK`
```json
[
  {
    "id": 101,
    "title": "Hoàn thành báo cáo",
    "description": "Viết báo cáo tiến độ dự án",
    "status": "todo",
    "priority": "high",
    "userId": 1
  }
]
```
Response lỗi gợi ý
Nếu user không tồn tại:
Status code: `404 Not Found`
```json
{
  "message": "User not found with id: 1"
}
```
---
4.11. Liệt kê toàn bộ công việc của một người dùng
Endpoint
```http
GET /users/{userId}/tasks
```
Mô tả
Lấy toàn bộ công việc đang được giao cho một người dùng cụ thể.
Đây là endpoint dạng nested resource vì task được liệt kê theo user.
Path variable
Tên	Kiểu dữ liệu	Mô tả
`userId`	Long	ID của người dùng cần lấy danh sách task
Response thành công
Status code: `200 OK`
```json
[
  {
    "id": 101,
    "title": "Hoàn thành báo cáo",
    "description": "Viết báo cáo tiến độ dự án",
    "status": "todo",
    "priority": "high",
    "userId": 1
  },
  {
    "id": 104,
    "title": "Chuẩn bị tài liệu họp",
    "description": "Tổng hợp nội dung cho buổi họp sprint",
    "status": "done",
    "priority": "medium",
    "userId": 1
  }
]
```
Response lỗi gợi ý
Nếu user không tồn tại:
Status code: `404 Not Found`
```json
{
  "message": "User not found with id: 1"
}
```
---
4.12. Gắn công việc cho người dùng
Endpoint
```http
POST /users/{userId}/tasks/{taskId}
```
Mô tả
Gắn một công việc đã tồn tại cho một người dùng đã tồn tại.
Endpoint này thể hiện thao tác tạo/cập nhật quan hệ giữa `User` và `Task`.
Có thể hiểu rằng server sẽ cập nhật:
```json
{
  "task.userId": 1
}
```
Path variable
Tên	Kiểu dữ liệu	Mô tả
`userId`	Long	ID người dùng được giao task
`taskId`	Long	ID công việc cần gắn cho user
Request body
Không bắt buộc request body vì cả `userId` và `taskId` đã nằm trên URL.
Response thành công
Status code: `200 OK`
```json
{
  "id": 101,
  "title": "Hoàn thành báo cáo",
  "description": "Viết báo cáo tiến độ dự án",
  "status": "todo",
  "priority": "high",
  "userId": 1
}
```
Response lỗi gợi ý
Nếu user không tồn tại:
Status code: `404 Not Found`
```json
{
  "message": "User not found with id: 1"
}
```
Nếu task không tồn tại:
Status code: `404 Not Found`
```json
{
  "message": "Task not found with id: 101"
}
```
---
5. Tổng hợp endpoint
   STT	Chức năng	Method	Endpoint
   1	Lấy toàn bộ công việc	`GET`	`/tasks`
   2	Lấy toàn bộ người dùng	`GET`	`/users`
   3	Tạo mới công việc	`POST`	`/tasks`
   4	Tạo mới người dùng	`POST`	`/users`
   5	Cập nhật trạng thái công việc	`PATCH`	`/tasks/{taskId}/status`
   6	Cập nhật vai trò người dùng	`PATCH`	`/users/{userId}/role`
   7	Xóa công việc	`DELETE`	`/tasks/{taskId}`
   8	Xóa người dùng	`DELETE`	`/users/{userId}`
   9	Tìm task priority high	`GET`	`/tasks?priority=high`
   10	Tìm task priority high của user id 1	`GET`	`/tasks?priority=high&userId=1`
   11	Liệt kê task của một user	`GET`	`/users/{userId}/tasks`
   12	Gắn task cho user	`POST`	`/users/{userId}/tasks/{taskId}`
---
6. Gợi ý mapping trong Spring Boot
   Các annotation chính có thể sử dụng:
   `@RestController`
   `@RequestMapping("/tasks")`
   `@RequestMapping("/users")`
   `@GetMapping`
   `@PostMapping`
   `@PatchMapping`
   `@DeleteMapping`
   `@RequestBody`
   `@PathVariable`
   `@RequestParam`
   `@Valid`
   Ví dụ mapping cho tạo mới task:
```java
@PostMapping("/tasks")
public ResponseEntity<TaskResponse> createTask(@Valid @RequestBody CreateTaskRequest request) {
    // validate userId nếu request có userId
    // tạo task
    // return 201 Created
}
```
Ví dụ mapping cho lọc task:
```java
@GetMapping("/tasks")
public ResponseEntity<List<TaskResponse>> getTasks(
        @RequestParam(required = false) String priority,
        @RequestParam(required = false) Long userId
) {
    // Nếu không có param: trả về toàn bộ task
    // Nếu có priority: lọc theo priority
    // Nếu có userId: lọc theo userId
    // Nếu có cả hai: lọc theo cả priority và userId
}
```
Ví dụ mapping cho gắn task cho user:
```java
@PostMapping("/users/{userId}/tasks/{taskId}")
public ResponseEntity<TaskResponse> assignTaskToUser(
        @PathVariable Long userId,
        @PathVariable Long taskId
) {
    // kiểm tra user tồn tại
    // kiểm tra task tồn tại
    // cập nhật task.userId = userId
    // return task đã cập nhật
}
```
---
7. Quy ước HTTP status code
   Status code	Ý nghĩa	Khi nào dùng
   `200 OK`	Thành công	Lấy dữ liệu, cập nhật thành công
   `201 Created`	Tạo mới thành công	Tạo mới user/task
   `204 No Content`	Thành công nhưng không trả body	Xóa user/task thành công
   `400 Bad Request`	Request không hợp lệ	Sai validation, sai enum
   `404 Not Found`	Không tìm thấy resource	Không tìm thấy user/task
   `409 Conflict`	Xung đột dữ liệu	Email trùng, xóa user còn task
---
8. Kết luận
   Thiết kế trên đảm bảo các yêu cầu:
   Endpoint dùng danh từ số nhiều: `/users`, `/tasks`
   Sử dụng đúng HTTP Method theo RESTful
   Có API tạo mới bằng `POST` và nhận JSON qua `@RequestBody`
   Có liên kết giữa `Task` và `User` thông qua `userId`
   Có endpoint lọc task theo `priority` và `userId`
   Có endpoint nested resource để liệt kê task của một user
   Có validation cơ bản và response lỗi rõ ràng