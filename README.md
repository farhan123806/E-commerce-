# E-commerce-
Creating a sample Backend Application in FastAPI and Python.
To design an e-commerce backend application using FastAPI and Python based on the provided requirements, we'll break down the implementation step by step.

Step 1: Setting up the Environment

First, create a new directory for your project and set up a Python virtual environment:

bash code

mkdir ecommerce_app
cd ecommerce_app
python -m venv venv
source venv/bin/activate  # Activate the virtual environment

Now, let's install the necessary libraries:

bash code

pip install fastapi uvicorn motor

Step 2: Creating the FastAPI Application

Create a FastAPI application and define the required models, endpoints, and routes.

python

# main.py

from fastapi import FastAPI, HTTPException, Query, Path
from typing import List, Optional
from pydantic import BaseModel
from motor.motor_asyncio import AsyncIOMotorClient

app = FastAPI()

# Initialize the MongoDB client
client = AsyncIOMotorClient("mongodb://localhost:27017")
db = client["ecommerce_db"]
products_collection = db["products"]
orders_collection = db["orders"]

# Define models
class Product(BaseModel):
    name: str
    price: float
    available_quantity: int

class UserAddress(BaseModel):
    city: str
    country: str
    zip_code: str

class OrderItem(BaseModel):
    productId: str
    boughtQuantity: int
    totalAmount: float

class Order(BaseModel):
    timestamp: str
    items: List[OrderItem]
    userAddress: UserAddress

# Create dummy products
@app.on_event("startup")
async def startup_db():
    await products_collection.insert_many([
        Product(name="TV", price=599.99, available_quantity=10).dict(),
        Product(name="Laptop", price=999.99, available_quantity=5).dict(),
        # Add more dummy products here
    ])

# API to List all available products
@app.get("/products/", response_model=List[Product])
async def list_products():
    products = await products_collection.find({}).to_list(None)
    return products

# API to Create a new order
@app.post("/orders/", response_model=dict)
async def create_order(order: Order):
    total_amount = sum(item.boughtQuantity * (product["price"]) for item, product in zip(order.items, await products_collection.find({}).to_list(None)))
    order_dict = order.dict()
    order_dict["totalAmount"] = total_amount
    result = await orders_collection.insert_one(order_dict)
    return {"orderId": str(result.inserted_id)}

# API to fetch all orders with pagination
@app.get("/orders/", response_model=List[dict])
async def fetch_orders(skip: int = Query(0, description="Number of records to skip"),
                       limit: int = Query(10, description="Maximum number of records to return")):
    orders = await orders_collection.find({}).skip(skip).limit(limit).to_list(None)
    return orders

# API to fetch a single order using Order ID
@app.get("/orders/{order_id}", response_model=dict)
async def fetch_single_order(order_id: str = Path(..., description="ID of the order")):
    order = await orders_collection.find_one({"_id": order_id})
    if order is None:
        raise HTTPException(status_code=404, detail="Order not found")
    return order

# API to update product quantity
@app.put("/products/{product_id}", response_model=dict)
async def update_product_quantity(product_id: str = Path(..., description="ID of the product"),
                                  quantity: int = Query(..., description="New available quantity")):
    result = await products_collection.update_one({"_id": product_id}, {"$set": {"available_quantity": quantity}})
    if result.modified_count == 0:
        raise HTTPException(status_code=404, detail="Product not found")
    return {"message": "Product quantity updated"}

    Step 3: Running the FastAPI Application

We can run the FastAPI application with the following command:

bash code

uvicorn main:app --host 0.0.0.0 --port 8000 --reload

The FastAPI application will be accessible at http://localhost:8000.

Step 4: MongoDB Setup

Start the MongoDB Server:

Once MongoDB is installed, we can start the server. Open a command prompt and run the following command:

For Linux/macOS:

bash code:

"C:\Program Files\MongoDB\Server\4.4\bin\mongod.exe" --dbpath "C:\data\db"
Make sure to replace the path with the appropriate location of your MongoDB installation and data directory.

Create a Database:

We can create a database for our e-commerce application. Open a new terminal window and run the MongoDB shell:

bash code:

mongo

Create a new database:

bash code:

use ecommerce_db

Now you have a database named ecommerce_db. You'll use this database for storing products, orders, and other data for your application.

Create Collections

MongoDB stores data in collections. You'll need two collections: products and orders.

Create a products collection:
bash code:

db.createCollection("products")

Create an orders collection:
bash code:

db.createCollection("orders")

Insert Dummy Data:

You can insert some dummy data into the products collection to get started. Use the MongoDB shell to insert sample product data:

bash code:

db.products.insertMany([
    {
        "name": "TV",
        "price": 499.99,
        "available_quantity": 10
    },
    {
        "name": "Laptop",
        "price": 999.99,
        "available_quantity": 5
    }
    // Add more products as needed
])

Configure FastAPI to Use MongoDB:

In your FastAPI application (assuming you've already implemented it based on the previous guidance), you can connect to your MongoDB database using Motor, an async driver for MongoDB.

You will need to replace the connection string and other MongoDB settings in your FastAPI application. Here's how you can do it:
from motor.motor_asyncio import AsyncIOMotorClient

# Connect to MongoDB
MONGODB_URI = "mongodb://localhost:27017"  # Update with your MongoDB connection string
client = AsyncIOMotorClient(MONGODB_URI)
db = client["ecommerce_db"]  # Use the same database name created earlier

# Define your collections
products_collection = db["products"]
orders_collection = db["orders"]

Now, you can use products_collection and orders_collection to interact with your MongoDB database in your FastAPI application.


