## 1. `@PathVariable`

Used when the value is part of the URL path.

### Request

```
GET /users/101
```

### Controller

```
@GetMapping("/users/{id}")public User getUser(@PathVariable Long id) {    return userService.getUser(id);}
```

### Value received

```
id = 101
```

### Typical use

When identifying a specific resource.

Examples:

```
GET /users/101GET /products/5DELETE /orders/20
```

---

## 2. `@RequestParam`

Used when the value comes from query parameters.

### Request

```
GET /users?id=101
```

### Controller

```
@GetMapping("/users")public User getUser(@RequestParam Long id) {    return userService.getUser(id);}
```

### Value received

```
id = 101
```

### Multiple parameters

```
GET /users?page=1&size=10&sort=name
```

```
@GetMapping("/users")public List<User> getUsers(        @RequestParam int page,        @RequestParam int size,        @RequestParam String sort) {    ...}
```

### Typical use

Filtering, sorting, pagination, searching.

Examples:

```
GET /products?category=mobileGET /users?page=1&size=20GET /employees?department=IT
```

---

## 3. `@RequestBody`

Used when data is sent in the body of the request, usually JSON.

### Request

```
POST /usersContent-Type: application/json{  "name": "Aneesh",  "age": 25}
```

### DTO

```
public class UserRequest {    private String name;    private int age;    // getters and setters}
```

### Controller

```
@PostMapping("/users")public String createUser(@RequestBody UserRequest request) {    return request.getName();}
```

### Value received

```
request.getName() = "Aneesh"request.getAge() = 25
```

### Typical use

Creating or updating complex objects.

Examples:

```
POST /usersPUT /users/101PATCH /users/101
```