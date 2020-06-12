# Banking API
The Banking API will manage the bank accounts of its users. It will be managed by the Bank's employees and admins. Employees and Admins count as Premium Users with additional abilities. Employees can view all customer information, but not modify in any way. Admins can both view all user information, as well as directly modify it. Standard users should be able to register and login to see their account information. They can have either Checking or Savings accounts. Savings accounts will have interest rates, that can be determined as you see fit. All users must be able to update their personal information, such as username, password, first and last names, as well as email. Accounts owned by users must support withdrawal, deposit, and transfer. Transfer of funds should be allowed between accounts owned by the same user, as well as between accounts owned by different users. Standard Users should be able to upgrade to Premium (the cost of such an upgrade may be freely determined). With this premium status, Users should now also be able to open new joint accounts (accounts with multiple owners), as well as be able to add users to pre-existing accounts.

## Models
The below models are an outline that supports the required features. They can be expanded or modified as needed.

### **User**
The User model keeps track of users information.
```java
public abstract class AbstractUser {
  private int userId; // primary key
  private String username; // not null, unique
  private String password; // not null
  private String firstName; // not null
  private String lastName; // not null
  private String email; // not null
  private Role role;
}
```

### **Role**
The Role model keeps track of user permissions. Can be expanded for more features later. Could be `Standard`, `Premium`, `Employee`, or `Admin`. Alternatively, this could be an Enum.

```java
public class Role {
  private int roleId; // primary key
  private String role; // not null, unique
}

public enum Role {
  Standard, Premium, Employee, Admin
}
```

### **Account**
The Account model is used to represent a single account for a user
```java
public abstract class AbstractAccount {
  private int accountId; // primary key
  private double balance;  // not null
  private AccountStatus status;
  private AccountType type;
}
```

### **AccountStatus**
The AccountStatus model is used to track the status of accounts. Status possibilities are `Pending`, `Open`, or `Closed`, or `Denied`. Alternatively, this could be an Enum.
```java
public class AccountStatus {
  private int statusId; // primary key
  private String status; // not null, unique
}

public enum AccountStatus {
  Pending, Open, Closed, Denied
}
```

### **AccountType**
The AccountType model is used to track what kind of account is being created. Type possibilities are `Checking` or `Savings`. Alternatively, this could be an Enum.
```java
public class AccountType {
  private int typeId; // primary key
  private String type; // not null, unique
}

public enum AccountType {
  Checking, Savings
}
```

# Endpoints

## Security
  Security should be handled through session storage.
  If a user does not have permission to access a particular endpoint it should return the following:
  * **Status Code:** 401 UNAUTHORIZED <br />
    **Content:**
    ```json
    {
      "message": "The incoming token has expired"
    }
    ```
    Occurs if they do not have the appropriate permissions.

## Available Endpoints

### **Login**
* **URL:** `/login`

* **Method:** `POST`

* **Request:**
  ```json
  {
    username: String,
    password: String
  }
  ```

* **Response:**
  ```json
  User
  ```

* **Error Response**
  * **Status Code:** 400 BAD REQUEST

  ```json
  {
    "message": "Invalid Credentials"
  }
  ```
### **Find Users**
* **URL:** `/users`

* **Method:** `GET`

* **Allowed Roles** `Employee` or `Admin`

* **Response:**
  ```json
  [
    User
  ]
  ```

### **Find Users By Id**
* **URL:** `/users/:id`

* **Method:** `GET`

* **Allowed Roles** `Employee` or `Admin` or if the id provided matches the id of the current user

* **Response:**
  ```json
  User
  ```

### **Update User**
* **URL:** `/users`

* **Method:** `PUT`

* **Allowed Roles** `Admin` or if the id provided matches the id of the current user

* **Request**
  ```json
  User
  ```

* **Response:**
  ```json
  User
  ```

### **Find Accounts**
* **URL:** `/accounts`

* **Method:** `GET`

* **Allowed Roles** `Employee` or `Admin`

* **Response:**
  ```json
  [
    Account
  ]
  ```

### **Find Accounts By Id**
* **URL:** `/accounts/:id`

* **Method:** `GET`

* **Allowed Roles** `Employee` or `Admin` or if the account belongs to the current user

* **Response:**
  ```json
  Account
  ```

### **Find Accounts By Status**
* **URL:** `/accounts/status/:statusId`

* **Method:** `GET`

* **Allowed Roles** `Employee` or `Admin`

* **Response:**
  ```json
  [
    Account
  ]
    ```

### **Find Accounts By User**
* **URL:** `/accounts/owner/:userId`
  For a challenge you could do this instead:
  `/accounts/owner/:userId?accountType=type

* **Method:** `GET`

* **Allowed Roles** `Employee` or `Admin` or if the id provided matches the id of the current user

* **Response:**
  ```json
  [
    Account
  ]
  ```

### **Submit Account**
* **URL:** `/accounts`

* **Method:** `POST`

* **Allowed Roles** `Employee` or `Admin` or if the account belongs to the current user

* **Request:**
  The accountId should be 0
  ```json
  Account
  ```

* **Response:**
  * **Status Code** 201 CREATED

  ```json
  Account
  ```


### **Update Account**
* **URL:** `/accounts`

* **Method:** `PUT`

* **Allowed Roles** `Admin`

* **Request**
  ```json
  Account
  ```

* **Response:**
  ```json
  Account
  ```

## RPC Endpoints
These endpoints are not RESTful, but are included to more conveniently simulate user actions

### **Withdraw**
* **URL:** `/accounts/withdraw`

* **Method:** `POST`

* **Allowed Roles** `Admin` or if the account belongs to the current user

* **Request**
  ```json
  {
    accountId: int,
    amount: double
  }
  ```

* **Response:**
  ```json
  {
    "message": "${amount} has been withdrawn from Account #{accountId}"
  }
  ```

### **Withdraw**
* **URL:** `/accounts/deposit`

* **Method:** `POST`

* **Allowed Roles** `Admin` or if the account belongs to the current user

* **Request**
  ```json
  {
    accountId: int,
    amount: double
  }
  ```

* **Response:**
  ```json
  {
    "message": "${amount} has been deposited to Account #{accountId}"
  }
  ```

### **Transfer**
* **URL:** `/accounts/transfer`

* **Method:** `POST`

* **Allowed Roles** `Admin` or if the source account belongs to the current user

* **Request**
  ```json
  {
    sourceAccountId: int,
    targetAccountId: int,
    amount: double
  }
  ```

* **Response:**
  ```json
  {
    "message": "${amount} has been transferred from Account #{sourceAccountId} to Account #{targetAccountId}"
  }
  ```

### **Pass Time**
This endpoint is designed to simulate the passing of time for Savings Accounts to accrue interest
* **URL:** `/passTime`

* **Method:** `POST`

* **Allowed Roles** `Admin`

* **Request**
  ```json
  {
    numOfMonths: int
  }
  ```

* **Response:**
  ```json
  {
    "message": "{numOfMonths} months of interest has been accrued for all Savings Accounts"
  }
  ```

# Stretch Goals
These are not part of the core requirements but are things that could be worked on once the core requirements are done.
  * Password Hashing
  * Paging and Sorting endpoints: [Reference For How](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design#filter-and-paginate-data)
  * Using JSON Web Tokens (JWTs) instead of Session Storage
