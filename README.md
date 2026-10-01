# WeCare-Beauty-product-management-system

[main program.py](https://github.com/user-attachments/files/32922008/main.program.py)
import os
from read import load_inventory, display_products, product_details
from operations import process_sale, restock_products, add_new_product

def main():
    """Main function to run the WeCare Beauty Product Management System"""
    print("\n===== WeCare Beauty Product Management System =====")
    print("Initializing inventory system...")
    
    file_path = "product.txt"
    
    # Check if product.txt exists
    if not os.path.exists(file_path):
        print("Error: product.txt not found!")
        # Create an empty file if it doesn't exist
        try:
            with open(file_path, "w") as file:
                pass
            print("Created empty product.txt file.")
        except Exception as e:
            print(f"Error creating product.txt: {e}")
            return
    
    # Load inventory
    products = load_inventory(file_path)
    
    while True:
        print("\n===== WeCare Beauty Product Management System =====")
        print("1. Display Available Products")
        print("2. View Product Details")
        print("3. Process Sale Transaction")
        print("4. Restock Products")
        print("5. Add New Product")
        print("6. Exit")
        
        try:
            choice = input("\nEnter your choice (1-6): ")
            
            if choice == '1':
                display_products(products)
            
            elif choice == '2':
                product_details(products)
                
            elif choice == '3':
                customer_name = input("Enter customer name: ")
                process_sale(products, file_path, customer_name)
            
            elif choice == '4':
                supplier_name = input("Enter supplier name: ")
                restock_products(products, file_path, supplier_name)
            
            elif choice == '5':
                add_new_product(products, file_path)
            
            elif choice == '6':
                print("Thank you for using WeCare Beauty Product Management System!")
                break
            
            else:
                print("Invalid choice. Please try again.")
                
        except Exception as e:
            print(f"An error occurred: {e}")
            print("Please try again.")

if __name__ == "__main__":
    main()[operations.py](https://github.com/user-attachments/files/32922019/operations.py)

from read import display_products, find_product_by_id
from write import save_inventory, generate_sale_invoice, generate_restock_invoice
from read import Product

def process_sale(products, file_path, customer_name):
    """Process a sales transaction with 'buy three get one free' policy"""
    if not products:
        print("\nNo products available for sale.")
        return
        
    cart = []
    total_amount = 0
    
    while True:
        display_products(products)
        choice = input("\nEnter product ID to add to cart (or 'done' to complete transaction): ")
        
        if choice.lower() == 'done':
            break
            
        try:
            product_id = int(choice)
            product = find_product_by_id(products, product_id)
            
            if not product:
                print("Invalid product ID. Please try again.")
                continue
                
            quantity = int(input(f"Enter quantity of {product.name} to purchase: "))
            
            if quantity <= 0:
                print("Quantity must be positive.")
                continue
                
            if product.quantity < quantity:
                print(f"Not enough stock. Only {product.quantity} available.")
                continue
            
            # Calculate free items based on the "buy three get one free" policy
            free_items = quantity // 3
            total_items = quantity + free_items
            
            if total_items > product.quantity:
                print(f"Not enough stock for free items. Only {product.quantity} available.")
                continue
            
            item_total = quantity * product.selling_price
            
            cart.append({
                'product': product,
                'quantity': quantity,
                'free_items': free_items,
                'item_total': item_total
            })
            
            total_amount += item_total
            
            print(f"Added {quantity} {product.name} to cart (+ {free_items} free)")
            
        except ValueError:
            print("Please enter a valid number.")
    
    if not cart:
        print("No items in cart. Transaction cancelled.")
        return
    
    # Process the sale and generate invoice
    if generate_sale_invoice(cart, customer_name, total_amount):
        # Save updated inventory
        save_inventory(products, file_path)
        print(f"Total Amount: ₹{total_amount:.2f}")
        print("Transaction completed successfully!")

def add_new_product(products, file_path):
    """Add a new product to the inventory"""
    try:
        print("\n===== Add New Product =====")
        name = input("Enter product name: ")
        brand = input("Enter brand name: ")
        quantity = int(input("Enter quantity: "))
        cost_price = float(input("Enter cost price: "))
        origin = input("Enter country of origin: ")
        
        new_product = Product(name, brand, quantity, cost_price, origin)
        products.append(new_product)
        save_inventory(products, file_path)
        
        print(f"Product '{name}' added successfully!")
        return new_product
    except ValueError as e:
        print(f"Error adding product: {e}")
        return None

def restock_products(products, file_path, supplier_name):
    """Restock existing products or add new ones"""
    restocked_items = []
    total_cost = 0
    
    while True:
        print("\nRestock Options:")
        print("1. Restock existing product")
        print("2. Add new product")
        print("3. Finish restocking")
        
        option = input("Choose an option (1-3): ")
        
        if option == '3':
            break
            
        elif option == '1':
            display_products(products)
            if not products:
                print("No products available. Please add new products.")
                continue
                
            try:
                product_id = int(input("Enter product ID to restock: "))
                product = find_product_by_id(products, product_id)
                
                if not product:
                    print("Invalid product ID. Please try again.")
                    continue
                    
                quantity = int(input(f"Enter quantity of {product.name} to restock: "))
                
                if quantity <= 0:
                    print("Quantity must be positive.")
                    continue
                
                update_price = input("Do you want to update the cost price? (y/n): ").lower()
                
                if update_price == 'y':
                    new_cost_price = float(input("Enter new cost price: "))
                    product.cost_price = new_cost_price
                    product.selling_price = new_cost_price * 3
                
                item_cost = quantity * product.cost_price
                
                restocked_items.append({
                    'product': product,
                    'quantity': quantity,
                    'item_cost': item_cost
                })
                
                total_cost += item_cost
                
                # Update inventory
                product.quantity += quantity
                
                print(f"Added {quantity} {product.name} to restock list")
                
            except ValueError:
                print("Please enter a valid number.")
                
        elif option == '2':
            new_product = add_new_product(products, file_path)
            if new_product:
                item_cost = new_product.quantity * new_product.cost_price
                
                restocked_items.append({
                    'product': new_product,
                    'quantity': new_product.quantity,
                    'item_cost': item_cost
                })
                
                total_cost += item_cost
                print(f"Added {new_product.quantity} {new_product.name} to restock list")
        
        else:
            print("Invalid option. Please try again.")
    
    if not restocked_items:
        print("No items restocked.")
        return
    
    # Generate restock invoice
    if generate_restock_invoice(restocked_items, supplier_name, total_cost):
        # Save updated inventory
        save_inventory(products, file_path)
        print(f"Total Cost: {total_cost:.2f}")
        print("Restocking completed successfully!")[read.py](https://github.com/user-attachments/files/32922021/read.py)
import os

class Product:
    def __init__(self, name, brand, quantity, cost_price, origin):
        self.name = name
        self.brand = brand
        self.quantity = int(quantity)
        self.cost_price = float(cost_price)
        self.origin = origin
        # Selling price is 200% markup (3x the cost price)
        self.selling_price = self.cost_price * 3

    def __str__(self):
        return f"{self.name} | {self.brand} | {self.quantity} | ₹{self.selling_price:.2f} | {self.origin}"

def load_inventory(file_path):
    """Load product inventory from the text file"""
    products = []
    try:
        if not os.path.exists(file_path):
            print(f"Error: {file_path} not found.")
            return products
            
        with open(file_path, "r") as file:
            for line in file:
                data = line.strip().split(",")
                if len(data) >= 5:
                    name = data[0].strip()
                    brand = data[1].strip()
                    quantity = int(data[2].strip())
                    cost_price = float(data[3].strip())
                    origin = data[4].strip()
                    products.append(Product(name, brand, quantity, cost_price, origin))
        print(f"Successfully loaded {len(products)} products from inventory.")
    except Exception as e:
        print(f"Error loading inventory: {e}")
    
    return products

def display_products(products):
    """Display all available products with selling prices"""
    if not products:
        print("\nNo products available in inventory.")
        return
        
    print("\n===== Available Products =====")
    print("ID | Product Name | Brand | Quantity | Selling Price | Origin")
    print("-------------------------------------------------------------------")
    for i, product in enumerate(products):
        print(f"{i+1} | {product}")
    print("-------------------------------------------------------------------")

def find_product_by_id(products, product_id):
    """Find a product by its ID number"""
    if 1 <= product_id <= len(products):
        return products[product_id - 1]
    return None

def product_details(products):
    """Show detailed information about a specific product"""
    if not products:
        print("\nNo products available in inventory.")
        return
        
    display_products(products)
    try:
        product_id = int(input("\nEnter product ID to view details: "))
        product = find_product_by_id(products, product_id)
        
        if not product:
            print("Invalid product ID.")
            return
            
        print("\n===== Product Details =====")
        print(f"Name: {product.name}")
        print(f"Brand: {product.brand}")
        print(f"Country of Origin: {product.origin}")
        print(f"Available Stock: {product.quantity}")
        print(f"Cost Price: {product.cost_price:.2f}")
        print(f"Selling Price: {product.selling_price:.2f}")
        print(f"Profit Margin: {(product.selling_price - product.cost_price):.2f}")
        
    except ValueError:
        print("Please enter a valid number.")
[write.py](https://github.com/user-attachments/files/32922025/write.py)
[product.txt](https://github.com/user-attachments/files/32922035/product.txt)
Vitamin C serum, Garnier, 200, 1000.0, France
Skin Cleanser, Cetaphil, 100, 280.0, Switzerland
Sunscreen, Aqualogica, 200, 700.0, India
eye cream, cerave, 5, 500.0, United Kingdom
