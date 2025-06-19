# Complete Examples

This page provides comprehensive examples showing how to use PySpringModel for both read operations and modifying operations with commit control in real-world scenarios.

## Basic User Management System

### Model Definition

```python
from py_spring_model import PySpringModel
from sqlmodel import Field
from typing import Optional
from datetime import datetime

class User(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    email: str = Field(unique=True)
    age: int = Field()
    status: str = Field(default="active")
    created_at: datetime = Field(default_factory=datetime.now)
    updated_at: datetime = Field(default_factory=datetime.now)

class UserProfile(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    user_id: int = Field(foreign_key="user.id")
    bio: str = Field(default="")
    avatar_url: str = Field(default="")
    preferences: dict = Field(default_factory=dict)
```

### Repository Definition

```python
from py_spring_model import CrudRepository, Query
from typing import Optional, List

class UserRepository(CrudRepository[int, User]):
    # Dynamic queries
    def find_by_email(self, email: str) -> Optional[User]: ...
    def find_by_name_and_status(self, name: str, status: str) -> Optional[User]: ...
    def find_all_by_status(self, status: str) -> List[User]: ...
    def find_all_by_age_and_status(self, age: int, status: str) -> List[User]: ...
    
    # Custom queries
    @Query("SELECT * FROM user WHERE age > {min_age} ORDER BY age DESC")
    def find_users_older_than(self, min_age: int) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE email LIKE '%{domain}%'")
    def find_users_by_email_domain(self, domain: str) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE created_at >= {start_date}")
    def find_users_created_after(self, start_date: datetime) -> List[User]: ...
    
    @Query("SELECT COUNT(*) as count FROM user WHERE status = {status}")
    def count_users_by_status(self, status: str) -> int: ...
    
    # Modifying operations with commit control
    @Query("INSERT INTO user (name, email, age, status) VALUES ({name}, {email}, {age}, {status}) RETURNING *", is_modifying=True)
    def create_user(self, name: str, email: str, age: int, status: str = "active") -> User: ...
    
    @Query("UPDATE user SET name = {name}, age = {age}, updated_at = NOW() WHERE id = {user_id} RETURNING *", is_modifying=True)
    def update_user(self, user_id: int, name: str, age: int) -> User: ...
    
    @Query("UPDATE user SET status = {status}, updated_at = NOW() WHERE id = {user_id}", is_modifying=True)
    def update_user_status(self, user_id: int, status: str) -> None: ...
    
    @Query("DELETE FROM user WHERE id = {user_id}", is_modifying=True)
    def delete_user(self, user_id: int) -> None: ...

class UserProfileRepository(CrudRepository[int, UserProfile]):
    def find_by_user_id(self, user_id: int) -> Optional[UserProfile]: ...
    def find_all_by_user_ids(self, user_ids: List[int]) -> List[UserProfile]: ...
    
    # Modifying operations
    @Query("INSERT INTO userprofile (user_id, bio, avatar_url) VALUES ({user_id}, {bio}, {avatar_url}) RETURNING *", is_modifying=True)
    def create_profile(self, user_id: int, bio: str = "", avatar_url: str = "") -> UserProfile: ...
    
    @Query("UPDATE userprofile SET bio = {bio}, avatar_url = {avatar_url} WHERE user_id = {user_id} RETURNING *", is_modifying=True)
    def update_profile(self, user_id: int, bio: str, avatar_url: str) -> UserProfile: ...
```

### Service Layer (Read and Write Operations)

```python
from typing import Optional, List, Dict
from datetime import datetime, timedelta

class UserService:
    user_repository: UserRepository
    profile_repository: UserProfileRepository
    
    # Read operations only
    def get_user_by_id(self, user_id: int) -> Optional[User]:
        return self.user_repository.find_by_id(user_id)
    
    def get_user_by_email(self, email: str) -> Optional[User]:
        return self.user_repository.find_by_email(email)
    
    def get_user_with_profile(self, user_id: int) -> Optional[Dict]:
        user = self.user_repository.find_by_id(user_id)
        if not user:
            return None
        
        profile = self.profile_repository.find_by_user_id(user_id)
        return {
            "user": user,
            "profile": profile
        }
    
    def get_active_users(self) -> List[User]:
        return self.user_repository.find_all_by_status("active")
    
    def get_users_by_age_range(self, min_age: int, max_age: int) -> List[User]:
        return self.user_repository.find_users_older_than(min_age)
    
    def get_recent_users(self, days: int = 7) -> List[User]:
        start_date = datetime.now() - timedelta(days=days)
        return self.user_repository.find_users_created_after(start_date)
    
    def get_user_statistics(self) -> Dict:
        active_count = self.user_repository.count_users_by_status("active")
        inactive_count = self.user_repository.count_users_by_status("inactive")
        
        return {
            "active_users": active_count,
            "inactive_users": inactive_count,
            "total_users": active_count + inactive_count
        }
    
    def search_users(self, query: str) -> List[User]:
        # Search by name or email
        users = self.user_repository.find_all_by_name_or_email(query, query)
        return users
    
    def get_users_by_domain(self, domain: str) -> List[User]:
        return self.user_repository.find_users_by_email_domain(domain)
    
    # Write operations
    def create_user(self, name: str, email: str, age: int, status: str = "active") -> User:
        return self.user_repository.create_user(name, email, age, status)
    
    def update_user(self, user_id: int, name: str, age: int) -> Optional[User]:
        if not self.user_repository.find_by_id(user_id):
            return None
        return self.user_repository.update_user(user_id, name, age)
    
    def deactivate_user(self, user_id: int) -> bool:
        if not self.user_repository.find_by_id(user_id):
            return False
        self.user_repository.update_user_status(user_id, "inactive")
        return True
    
    def delete_user(self, user_id: int) -> bool:
        if not self.user_repository.find_by_id(user_id):
            return False
        self.user_repository.delete_user(user_id)
        return True
    
    def create_user_with_profile(self, name: str, email: str, age: int, bio: str = "", avatar_url: str = "") -> Dict:
        # Create user first
        user = self.user_repository.create_user(name, email, age)
        # Create profile
        profile = self.profile_repository.create_profile(user.id, bio, avatar_url)
        return {
            "user": user,
            "profile": profile
        }
```

### REST Controller (Full CRUD Operations)

```python
from py_spring_core import RestController, Get, Post, Put, Delete, PathVariable, RequestBody
from typing import List, Optional

@RestController("/api/users")
class UserController:
    user_service: UserService
    
    # Read operations
    @Get("/")
    def get_all_users(self) -> List[User]:
        return self.user_service.get_active_users()
    
    @Get("/{user_id}")
    def get_user(self, user_id: int) -> Optional[User]:
        return self.user_service.get_user_by_id(user_id)
    
    @Get("/{user_id}/profile")
    def get_user_with_profile(self, user_id: int) -> Optional[dict]:
        return self.user_service.get_user_with_profile(user_id)
    
    @Get("/search")
    def search_users(self, query: str) -> List[User]:
        return self.user_service.search_users(query)
    
    @Get("/statistics")
    def get_statistics(self) -> dict:
        return self.user_service.get_user_statistics()
    
    # Write operations
    @Post("/")
    def create_user(self, @RequestBody user_data: dict) -> User:
        return self.user_service.create_user(**user_data)
    
    @Post("/{user_id}/profile")
    def create_user_profile(self, user_id: int, @RequestBody profile_data: dict) -> dict:
        return self.user_service.create_user_with_profile(
            user_id=user_id, **profile_data
        )
    
    @Put("/{user_id}")
    def update_user(self, user_id: int, @RequestBody user_data: dict) -> Optional[User]:
        return self.user_service.update_user(user_id, **user_data)
    
    @Put("/{user_id}/deactivate")
    def deactivate_user(self, user_id: int) -> dict:
        success = self.user_service.deactivate_user(user_id)
        return {"success": success}
    
    @Delete("/{user_id}")
    def delete_user(self, user_id: int) -> dict:
        success = self.user_service.delete_user(user_id)
        return {"success": success}
```

## E-commerce System Example

### Models

```python
from py_spring_model import PySpringModel
from sqlmodel import Field
from typing import Optional, List
from datetime import datetime
from decimal import Decimal

class Product(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    description: str = Field()
    price: Decimal = Field()
    category: str = Field()
    stock_quantity: int = Field()
    is_active: bool = Field(default=True)
    created_at: datetime = Field(default_factory=datetime.now)

class Order(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    user_id: int = Field(foreign_key="user.id")
    total_amount: Decimal = Field()
    status: str = Field(default="pending")
    created_at: datetime = Field(default_factory=datetime.now)
    updated_at: datetime = Field(default_factory=datetime.now)

class OrderItem(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    order_id: int = Field(foreign_key="order.id")
    product_id: int = Field(foreign_key="product.id")
    quantity: int = Field()
    unit_price: Decimal = Field()
    total_price: Decimal = Field()
```

### Repositories

```python
from py_spring_model import CrudRepository, Query
from typing import Optional, List

class ProductRepository(CrudRepository[int, Product]):
    def find_by_category(self, category: str) -> List[Product]: ...
    def find_by_name_and_category(self, name: str, category: str) -> Optional[Product]: ...
    def find_all_by_is_active(self, is_active: bool) -> List[Product]: ...
    
    @Query("SELECT * FROM product WHERE price BETWEEN {min_price} AND {max_price}")
    def find_by_price_range(self, min_price: Decimal, max_price: Decimal) -> List[Product]: ...
    
    @Query("SELECT * FROM product WHERE stock_quantity > 0 AND is_active = true")
    def find_available_products(self) -> List[Product]: ...
    
    @Query("SELECT category, COUNT(*) as count FROM product GROUP BY category")
    def get_product_count_by_category(self) -> List[dict]: ...
    
    # Modifying operations
    @Query("INSERT INTO product (name, description, price, category, stock_quantity) VALUES ({name}, {description}, {price}, {category}, {stock_quantity}) RETURNING *", is_modifying=True)
    def create_product(self, name: str, description: str, price: Decimal, category: str, stock_quantity: int) -> Product: ...
    
    @Query("UPDATE product SET price = {price}, stock_quantity = {stock_quantity} WHERE id = {product_id} RETURNING *", is_modifying=True)
    def update_product(self, product_id: int, price: Decimal, stock_quantity: int) -> Product: ...
    
    @Query("UPDATE product SET is_active = false WHERE id = {product_id}", is_modifying=True)
    def deactivate_product(self, product_id: int) -> None: ...

class OrderRepository(CrudRepository[int, Order]):
    def find_by_user_id(self, user_id: int) -> List[Order]: ...
    def find_by_status(self, status: str) -> List[Order]: ...
    def find_by_user_id_and_status(self, user_id: int, status: str) -> List[Order]: ...
    
    @Query("SELECT * FROM `order` WHERE created_at >= {start_date} AND created_at <= {end_date}")
    def find_orders_by_date_range(self, start_date: datetime, end_date: datetime) -> List[Order]: ...
    
    @Query("SELECT SUM(total_amount) as total FROM `order` WHERE status = {status}")
    def get_total_amount_by_status(self, status: str) -> Decimal: ...
    
    # Modifying operations
    @Query("INSERT INTO `order` (user_id, total_amount, status) VALUES ({user_id}, {total_amount}, {status}) RETURNING *", is_modifying=True)
    def create_order(self, user_id: int, total_amount: Decimal, status: str = "pending") -> Order: ...
    
    @Query("UPDATE `order` SET status = {status}, updated_at = NOW() WHERE id = {order_id} RETURNING *", is_modifying=True)
    def update_order_status(self, order_id: int, status: str) -> Order: ...

class OrderItemRepository(CrudRepository[int, OrderItem]):
    def find_by_order_id(self, order_id: int) -> List[OrderItem]: ...
    def find_by_product_id(self, product_id: int) -> List[OrderItem]: ...
    
    # Modifying operations
    @Query("INSERT INTO orderitem (order_id, product_id, quantity, unit_price, total_price) VALUES ({order_id}, {product_id}, {quantity}, {unit_price}, {total_price}) RETURNING *", is_modifying=True)
    def create_order_item(self, order_id: int, product_id: int, quantity: int, unit_price: Decimal, total_price: Decimal) -> OrderItem: ...
```

### Service Layer (E-commerce Operations)

```python
from decimal import Decimal
from typing import List, Optional, Dict

class ProductService:
    product_repository: ProductRepository
    
    def get_available_products(self) -> List[Product]:
        return self.product_repository.find_available_products()
    
    def get_products_by_category(self, category: str) -> List[Product]:
        return self.product_repository.find_by_category(category)
    
    def get_products_by_price_range(self, min_price: Decimal, max_price: Decimal) -> List[Product]:
        return self.product_repository.find_by_price_range(min_price, max_price)
    
    def get_product_statistics(self) -> Dict:
        return self.product_repository.get_product_count_by_category()

class OrderService:
    order_repository: OrderRepository
    order_item_repository: OrderItemRepository
    product_repository: ProductRepository
    
    def get_user_orders(self, user_id: int) -> List[Order]:
        return self.order_repository.find_by_user_id(user_id)
    
    def get_order_with_items(self, order_id: int) -> Optional[Dict]:
        order = self.order_repository.find_by_id(order_id)
        if not order:
            return None
        
        items = self.order_item_repository.find_by_order_id(order_id)
        return {
            "order": order,
            "items": items
        }
    
    def get_orders_by_date_range(self, start_date: datetime, end_date: datetime) -> List[Order]:
        return self.order_repository.find_orders_by_date_range(start_date, end_date)
    
    def get_order_statistics(self) -> Dict:
        pending_total = self.order_repository.get_total_amount_by_status("pending")
        completed_total = self.order_repository.get_total_amount_by_status("completed")
        cancelled_total = self.order_repository.get_total_amount_by_status("cancelled")
        
        return {
            "pending_total": pending_total or Decimal('0'),
            "completed_total": completed_total or Decimal('0'),
            "cancelled_total": cancelled_total or Decimal('0')
        }
```

## Application Configuration

### app-config.json

```json
{
  "database": {
    "url": "sqlite:///./app.db",
    "echo": false
  },
  "server": {
    "host": "localhost",
    "port": 8000
  }
}
```

### Main Application

```python
from py_spring_core import PySpringApplication
from py_spring_model.py_spring_model_provider import provide_py_spring_model

def main():
    app = PySpringApplication(
        "./app-config.json",
        entity_providers=[provide_py_spring_model()]
    )
    app.run()

if __name__ == "__main__":
    main()
```

## Important Notes

- **Full CRUD Support**: PySpringModel now supports both read and write operations
- **Transaction Control**: Use `is_modifying=True` for INSERT, UPDATE, DELETE operations
- **Commit Management**: Fine-grained control over when database changes are committed
- **Batch Operations**: Manual transaction control for efficient bulk operations
- **Data Integrity**: Proper transaction handling ensures data consistency
- **Backward Compatibility**: All existing read operations continue to work unchanged 