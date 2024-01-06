# Day 1 Learning

## Overview

Today's learning focused on setting up a RESTful API using C# with MySQL, implementing CRUD operations for a user entity.

## Project Structure

```
6124gpt/
│
├── Helpers/
│   ├── IDatabaseHelper.cs
│   ├── EncryptionHelper.cs
│   └── DatabaseHelper.cs
│
├── Models/
│   └── UserModel.cs
│
├── Controllers/
│   └── UserController.cs
│
├── Program.cs
├── Startup.cs
```

## Key Learnings

### 1. Project Setup
### Required Packeges

```cmd
Install-Package Microsoft.AspNetCore.Mvc.NewtonsoftJson
Install-Package Swashbuckle.AspNetCore

```
- Created a new C# project named `6124gpt` in Visual Studio 2022.
- Structured the project with folders for `Helpers`, `Models`, and `Controllers`.

### 2. User Entity

- Defined a simple `UserModel` with properties for `UserId`, `UserName`, `Email`, and `Password`.

### 3. Database Connectivity

- Implemented a `DatabaseHelper` class to handle database operations.
- Established a connection to MySQL using a connection string.
- Implemented methods for CRUD operations (GetUsers, GetUserById, CreateUser, UpdateUser, DeleteUser).
- Included `IDatabaseHelper` interface to define the contract for database operations.

### 4. Password Encryption

- Created an `EncryptionHelper` class for hashing passwords using SHA-1.

### 5. UserController

- Implemented a `UserController` with CRUD operations.
- Utilized dependency injection for `DatabaseHelper` in `UserController`.
- Handled HTTP requests for listing users, getting a user by ID, creating a new user, updating a user, and deleting a user.

### 6. Swagger Documentation

- Integrated Swagger for API documentation.

## Code Snippets

### `Models/UserModel.cs`

```csharp
namespace _6124Gpt.Models
{
    public class UserModel
    {
        public int UserId { get; set; }
        public string UserName { get; set; }
        public string Email { get; set; }
        public string Password { get; set; }
    }
}

```

### `Helpers/IDatabaseHelper.cs`

```csharp
using _6124Gpt.Models;
using System.Collections.Generic;

namespace _6124Gpt.Helpers
{
    public interface IDatabaseHelper
    {
        List<UserModel> GetUsers();
        UserModel GetUserById(int userId);
        UserModel CreateUser(UserModel newUser);
        void UpdateUser(UserModel updatedUser);
        void DeleteUser(int userId);
    }
}

```

### `Helpers/EncryptionHelper.cs`

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

namespace _6124Gpt.Helpers
{
    public class EncryptionHelper : IDisposable
    {
        private SHA1 sha1;

        public EncryptionHelper()
        {
            sha1 = SHA1.Create();
        }

        public string ComputeSHA1Hash(string input)
        {
            byte[] inputBytes = Encoding.UTF8.GetBytes(input);
            byte[] hashBytes = sha1.ComputeHash(inputBytes);
            string hashHex = BitConverter.ToString(hashBytes).Replace("-", "").ToLower();
            return hashHex;
        }

        public void Dispose()
        {
            sha1.Dispose();
        }
    }
}

```

### `Helpers/DatabaseHelper.cs`

```csharp
using MySql.Data.MySqlClient;
using _6124Gpt.Models;
using System;
using System.Collections.Generic;
using System.Text;

namespace _6124Gpt.Helpers
{
    public class DatabaseHelper : IDatabaseHelper
    {
        private readonly string connectionString = "Server=localhost;Port=3306;Database=company;User=root;Password=Dell;";
        public List<UserModel> GetUsers()
        {
            List<UserModel> users = new List<UserModel>();

            using (MySqlConnection connection = new MySqlConnection(connectionString))
            {
                connection.Open();
                string query = "SELECT * FROM users";

                using (MySqlCommand cmd = new MySqlCommand(query, connection))
                {
                    using (MySqlDataReader reader = cmd.ExecuteReader())
                    {
                        while (reader.Read())
                        {
                            UserModel user = new UserModel
                            {
                                UserId = Convert.ToInt32(reader["user_id"]),
                                UserName = reader["username"].ToString(),
                                Email = reader["email"].ToString(),
                                Password = reader["password"].ToString()
                            };

                            users.Add(user);
                        }
                    }
                }
            }

            return users;
        }

        public UserModel GetUserById(int userId)
        {
            using (MySqlConnection connection = new MySqlConnection(connectionString))
            {
                connection.Open();
                string query = "SELECT * FROM users WHERE user_id = @userId";

                using (MySqlCommand cmd = new MySqlCommand(query, connection))
                {
                    cmd.Parameters.AddWithValue("@userId", userId);

                    using (MySqlDataReader reader = cmd.ExecuteReader())
                    {
                        if (reader.Read())
                        {
                            return new UserModel
                            {
                                UserId = Convert.ToInt32(reader["user_id"]),
                                UserName = reader["username"].ToString(),
                                Email = reader["email"].ToString(),
                                Password = reader["password"].ToString()
                            };
                        }
                    }
                }
            }

            return null;
        }

        public UserModel CreateUser(UserModel newUser)
        {
            using (MySqlConnection connection = new MySqlConnection(connectionString))
            {
                connection.Open();
                string query = "INSERT INTO users (username, email, password) VALUES (@username, @email, @password)";

                using (MySqlCommand cmd = new MySqlCommand(query, connection))
                {
                    cmd.Parameters.AddWithValue("@username", newUser.UserName);
                    cmd.Parameters.AddWithValue("@email", newUser.Email);

                    // Hash the password using SHA-1
                    string hashedPassword = HashPassword(newUser.Password);
                    cmd.Parameters.AddWithValue("@password", hashedPassword);

                    cmd.ExecuteNonQuery();
                }
            }

            return newUser;
        }

        public void UpdateUser(UserModel updatedUser)
        {
            using (MySqlConnection connection = new MySqlConnection(connectionString))
            {
                connection.Open();
                string query = "UPDATE users SET username = @username, email = @email, password = @password WHERE user_id = @userId";

                using (MySqlCommand cmd = new MySqlCommand(query, connection))
                {
                    cmd.Parameters.AddWithValue("@username", updatedUser.UserName);
                    cmd.Parameters.AddWithValue("@email", updatedUser.Email);
                    cmd.Parameters.AddWithValue("@password", updatedUser.Password);
                    cmd.Parameters.AddWithValue("@userId", updatedUser.UserId);

                    cmd.ExecuteNonQuery();
                }
            }
        }

        public void DeleteUser(int userId)
        {
            using (MySqlConnection connection = new MySqlConnection(connectionString))
            {
                connection.Open();
                string query = "DELETE FROM users WHERE user_id = @userId";

                using (MySqlCommand cmd = new MySqlCommand(query, connection))
                {
                    cmd.Parameters.AddWithValue("@userId", userId);

                    cmd.ExecuteNonQuery();
                }
            }
        }

        // Helper method to hash the password using SHA-1
        private string HashPassword(string password)
        {
            using (EncryptionHelper encryptionHelper = new EncryptionHelper())
            {
                return encryptionHelper.ComputeSHA1Hash(password);
            }
        }
    }
}

```

### `Controllers/UserController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using _6124Gpt.Helpers;
using _6124Gpt.Models;
using System.Collections.Generic;

namespace _6124Gpt.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class UserController : ControllerBase
    {
        private readonly IDatabaseHelper _databaseHelper;

        public UserController(IDatabaseHelper databaseHelper)
        {
            _databaseHelper = databaseHelper;
        }

        [HttpGet]
        public ActionResult<IEnumerable<UserModel>> Get()
        {
            var users = _databaseHelper.GetUsers();
            return Ok(users);
        }

        [HttpGet("{id}")]
        public ActionResult<UserModel> GetById(int id)
        {
            var user = _databaseHelper.GetUserById(id);

            if (user == null)
            {
                return NotFound(); // HTTP 404 Not Found
            }

            return Ok(user);
        }

        [HttpPost]
        public ActionResult<UserModel> Create(UserModel newUser)
        {
            // Validation or other logic as needed

            var createdUser = _databaseHelper.CreateUser(newUser);

            return CreatedAtAction(nameof(GetById), new { id = createdUser.UserId }, createdUser);
        }

        [HttpPut("{id}")]
        public ActionResult<UserModel> Update(int id, UserModel updatedUser)
        {
            var existingUser = _databaseHelper.GetUserById(id);

            if (existingUser == null)
            {
                return NotFound(); // HTTP 404 Not Found
            }

            // Update properties of existingUser with values from updatedUser
            existingUser.UserName = updatedUser.UserName;
            existingUser.Email = updatedUser.Email;
            existingUser.Password = updatedUser.Password;

            _databaseHelper.UpdateUser(existingUser);

            return Ok(existingUser);
        }

        [HttpDelete("{id}")]
        public ActionResult Delete(int id)
        {
            var existingUser = _databaseHelper.GetUserById(id);

            if (existingUser == null)
            {
                return NotFound(); // HTTP 404 Not Found
            }

            _databaseHelper.DeleteUser(id);

            return NoContent(); // HTTP 204 No Content
        }
    }
}

```

### `Startup.cs`

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using _6124Gpt.Helpers;

namespace _6124Gpt
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();

            // Register the IDatabaseHelper and EncryptionHelper implementations
            services.AddScoped<IDatabaseHelper, DatabaseHelper>();
            services.AddScoped<EncryptionHelper>();

            // Enable Swagger/OpenAPI
            services.AddSwaggerGen();
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                app.UseSwagger();
                app.UseSwaggerUI();
            }

            app.UseHttpsRedirection();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}

```

### `Program.cs`

```csharp
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;

namespace _6124Gpt
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }
}

```

## Next Steps

- Explore further improvements like input validation, exception handling, and enhancing security measures.
- Consider additional features like user authentication and authorization.

