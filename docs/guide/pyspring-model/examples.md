# Complete Examples

**Note**: PySpringModel currently only supports **read operations**. All examples in this page focus on querying and reading data from the database.

This page provides comprehensive examples showing how to use PySpringModel for data retrieval in real-world scenarios.

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

class UserProfileRepository(CrudRepository[int, UserProfile]):
    def find_by_user_id(self, user_id: int) -> Optional[UserProfile]: ...
    def find_all_by_user_ids(self, user_ids: List[int]) -> List[UserProfile]: ...
```

### Service Layer (Read Operations Only)

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
```

### REST Controller (Read Operations Only)

```python
from py_spring_core import RestController, Get, PathVariable
from typing import List, Optional

@RestController("/api/users")
class UserController:
    user_service: UserService
    
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

class OrderRepository(CrudRepository[int, Order]):
    def find_by_user_id(self, user_id: int) -> List[Order]: ...
    def find_by_status(self, status: str) -> List[Order]: ...
    def find_by_user_id_and_status(self, user_id: int, status: str) -> List[Order]: ...
    
    @Query("SELECT * FROM `order` WHERE created_at >= {start_date} AND created_at <= {end_date}")
    def find_orders_by_date_range(self, start_date: datetime, end_date: datetime) -> List[Order]: ...
    
    @Query("SELECT SUM(total_amount) as total FROM `order` WHERE status = {status}")
    def get_total_amount_by_status(self, status: str) -> Decimal: ...

class OrderItemRepository(CrudRepository[int, OrderItem]):
    def find_by_order_id(self, order_id: int) -> List[OrderItem]: ...
    def find_by_product_id(self, product_id: int) -> List[OrderItem]: ...
```

### Service Layer (Read Operations Only)

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

- **Read-Only Operations**: All examples focus on data retrieval and querying
- **No Data Modification**: PySpringModel does not support creating, updating, or deleting data
- **SQLAlchemy for Writes**: For write operations, use SQLAlchemy directly
- **Future Enhancements**: Write operations are planned for future releases
- **Data Integrity**: Since only read operations are supported, data integrity is maintained 