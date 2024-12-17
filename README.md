#include <iostream>
#include <string>
#include <vector>
#include <ctime>
#include <algorithm>

using namespace std;

class Review; 
class Customer;

class Category {
private:
    int id;
    string name;
    string description;

public:
    Category(int id, const string& name, const string& description)
        : id(id), name(name), description(description) {}

    // Getters
    int getId() const { return id; }
    string getName() const { return name; }
    string getDescription() const { return description; }

    // Setters
    void setName(const string& name) { this->name = name; }
    void setDescription(const string& description) { this->description = description; }

    // Display category info
    void display() const {
        cout << "Category ID: " << id << ", Name: " << name << ", Description: " << description << endl;
    }
};

class Product {
private:
    int id;
    string name;
    string description;
    double price;
    int stock;
    vector<Review*> reviews;
    Category* category;

public:
    Product() : id(0), name(""), description(""), price(0), stock(0), reviews(0), category(nullptr) {}

    Product(const string& name, const string& description, double price, int stock, Category* category)
        : name(name), description(description), price(price), stock(stock), category(category) {}

    // getters
    int getId() const { return id; }
    string getName() const { return name; }
    string getDescription() const { return description; }
    double getPrice() const { return price; }
    int getStock() const { return stock; }
    vector<Review*> getReviews() const { return reviews; }
    Category* getCategory() const { return category; }

    //setters
    void setName(const std::string& name) { this->name = name; }
    void setDescription(const std::string& description) { this->description = description; }
    void setPrice(double price) { this->price = price; }
    void setStock(int stock) { this->stock = stock; }
    void setCategory(Category* category) { this->category = category; }

    //display info
    void display() {
        cout << id << " " << name << " " << description << " " << price << " " << stock << endl;
        if (category) {
            cout << "Category: ";
            category->display();
        }
    }


    //add review 
    void addReview(const std::string& customer, int rating, const std::string& comment) { 
        for (const Review* review : reviews) {
            //checking quantity of reviews by user
            if (review->getCustomer()->getName() == customer) { 
                cout << "Пользователь уже оставил отзыв на этот товар" << endl;
                return;
            }
        }
        cout << "Отзыв от пользователя " << customer << " не найден." << endl;
    }

    void printReviews() {
        cout << "Отзывы к товару " << name << ": " << endl;
        for (const Review* review : reviews) {
            cout << "Пользователь: " << review->getCustomer()->getName() << " Рейтинг: " << review->getRating() << " Отзыв: " << review->getComment() << endl; // Исправлено для доступа к имени клиента через указатель
        }
    }
};

class ProductCatalog {
private:
    vector<Product> products;

public:
    void addProduct(const Product& product) {
        products.push_back(product);
    }

    vector<Product> searchByName(const std::string& name) const {
        vector<Product> result;
        for (const auto& product : products) {
            if (product.getName().find(name) != string::npos) {
                result.push_back(product);
            }
        }
        return result;
    }

    vector<Product> searchByName(const std::string& description) const {
        vector<Product> result;
        for (const auto& product : products) {
            if (product.getDescription().find(description) != string::npos) {
                result.push_back(product);
            }
        }
        return result;
    }

    vector<Product> filterByPrice(double minPrice, double maxPrice) const {
        vector<Product> result;
        for (const auto& product : products) {
            if (product.getPrice() >= minPrice && product.getPrice() <= maxPrice) {
                result.push_back(product);
            }
        }
        return result;
    }

    void displayProducts(const vector<Product>& products) const {
        for (const auto& product : products) {
            std::cout << "ID: " << product.getId()
                << ", Name: " << product.getName()
                << ", Description: " << product.getDescription()
                << ", Price: " << product.getPrice()
                << ", Stock: " << product.getStock()
                << ", Reviews: " << product.getReviews().size() << std::endl;
        }
    }

};




class Notification {
private:
    int id;
    Customer* customer;
    string message;
    time_t date;

public:
    Notification() : id(0), customer(nullptr), message(""), date(0) {} // не шарю как по английски будет, но это конструктор, который стоит по умолчанию

    Notification(int id, Customer* customer, const std::string& message, time_t date) 
        : id(id), customer(customer), message(message), date(date) {}

    //getters
    int getId() const { return id; }
    Customer* getCustomer() const { return customer; }
    string getMessage() const { return message; }
    time_t getDate() const { return date; }

    //setters
    void setMessage(const std::string& message) { this->message = message; }
    void setDate(time_t date) { this->date = date; }
};

class Observer {
public:
    //virtual method that will be overriden
    virtual void update(const Notification& notification) = 0;
};

class Subject {
private:
    vector<Observer*> observers;

public:
    //add observer
    void addObserver(Observer* observer) {
        observers.push_back(observer);
    }

    //remove observer
    void removeObserver(Observer* observer) {
        observers.erase(remove(observers.begin(), observers.end(), observer), observers.end());
    }

    //notify all observers
    void notifyObservers(const Notification& notification) {
        for (Observer* observer : observers) {
            observer->update(notification);
        }

    }
};

class Customer : public Observer {
private:
    int id;
    string name;
    string email;
    vector<Product*> cart;

public:
    Customer(int id, const string& name, const string& email)
        : id(id), name(name), email(email) {}

    // getters
    int getId() const { return id; }
    string getName() const { return name; }
    string getEmail() const { return email; }
    vector<Product*> getCart() const { return cart; }

    //setters
    void setName(const std::string& name) { this->name = name; }
    void setEmail(const std::string& email) { this->email = email; }

    //remove products from cart
    void removeFromCart(Product* product) {
        cart.erase(remove(cart.begin(), cart.end(), product), cart.end()); 
    }

    void update(const Notification& notification) override {
        cout << "Покупатель " << name << ". принятое уведомление: " << notification.getMessage() << endl;
    }

    //display info
    void display() {
        cout << id << " " << name << " " << email << endl; 
        cout << "Корзина: " << endl;
        for (const auto& product : cart) {
            product->display();
        }
    }
};

class Order {
private:
    int id;
    Customer* customer;
    vector<Product*> products;
    double totalPrice;
    const Discount* discount;
    OrderStatus* status;

public:
    Order(int id, Customer* customer, const vector<Product*> products, double totalPrice, OrderStatus* status)
        : id(id), customer(customer), products(products), totalPrice(totalPrice), status(status) {}

    //calculating total price with discount
    double calculateTotalPrice() const {
        if (discount && discount->isValid(std::time(nullptr))) {
            double discountPrice = totalPrice * (discount->getPercentage() / 100);
            return totalPrice - discountPrice;
        }
        return totalPrice;
    }

    // getters
    int getId() const { return id; }
    Customer* getCustomer() const { return customer; }
    vector<Product*> getProducts() const { return products; }
    double genTotalPrice() const { return totalPrice; }

    //display info about order
    void display() const {
        cout << id << " " << customer->getName() << " " << totalPrice << " " << discount << endl; 
        cout << "Товары: " << endl;
        for (const auto& product : products) {
            product->display();
        }
    }
};

class OrderStatus {
private:
    int id;
    string name;
    string description;

public:
    OrderStatus(int id, const string& name, const string& description)
        : id(id), name(name), description(description) {}

    // Getters
    int getId() const { return id; }
    string getName() const { return name; }
    string getDescription() const { return description; }

    // Display status info
    void display() const {
        cout << "Status ID: " << id << ", Name: " << name << ", Description: " << description << endl;
    }
};

class Discount {
private:
    int id;
    double percentage;
    time_t startDate;
    time_t endDate;

public:
    Discount(double percentage, std::time_t startDate, std::time_t endDate)
        : percentage(percentage), startDate(startDate), endDate(endDate) {}

    //checking discount activity
    bool isValid(std::time_t currentDate) const {
        return currentDate >= startDate && currentDate <= endDate;
    }

    //getters
    int getId() const { return id; }
    double getPercentage() const { return percentage; }
    time_t getStartDate() const { return startDate; }
    time_t getEndDate() const { return endDate; }

    //setters
    void setPercentage(double percentage) { this->percentage = percentage; }
    void setStartDate(const std::time_t startDate) { this->startDate = startDate; }
    void setEndDate(const std::time_t endDate) { this->endDate = endDate; }
};

class Review {
private:
    Customer* customer;
    int rating;
    string comment;

public:
    Review(Customer* customer, int rating, const std::string& comment)
        : customer(customer), rating(rating), comment(comment) {}

    //getters
    Customer* getCustomer() const { return customer; }
    int getRating() const { return rating; }
    string getComment() const { return comment; }

    //setters
    void setRating(int rating) { this->rating = rating; }
    void setComment(const std::string& comment) { this->comment = comment; } 
};

class OnlineStore {
private:
    vector<Product*> products;
    vector<Customer*> customers;
    vector<Order*> orders;
    vector<Category*> categories;
    vector<OrderStatus*> orderStatuses;

public:
    //add product
    void addProduct(Product* product) {
        products.push_back(product);
    }

    //remove product
    void removeProduct(Product* product) {
        products.erase(remove(products.begin(), products.end(), product), products.end());
    }

    //edit product
    void editProduct(Product* product, const std::string& name, const std::string& description, double price, int stock) {
        product->setName(name);
        product->setDescription(description);
        product->setPrice(price);
        product->setStock(stock);
    }

    //add customer to store
    void addCustomer(Customer* customer) {
        customers.push_back(customer);
    }

    //remove customer from store
    void removeCustomer(Customer* customer) {
        customers.erase(remove(customers.begin(), customers.end(), customer), customers.end());
    }

    //create order
    void createOrder(Customer* customer) {
        std::vector<Product*> cart = customer->getCart();
        if (!cart.empty()) {
            double totalPrice = 0;
            for (const auto& product : cart) {
                totalPrice += product->getPrice();
            }

            OrderStatus* defaultStatus = new OrderStatus(1, "В обработке", "Ваш заказ находится в обработке.");

            Order* order = new Order(orders.size() + 1, customer, cart, totalPrice, defaultStatus);
            orders.push_back(order);
            customer->getCart().clear();
        }
        else {
            cout << "Корзина пуста. Не удается создать заказ." << endl;
        }
    }

    // Add category
    void addCategory(Category* category) {
        categories.push_back(category);
    }

    // Remove category
    void removeCategory(Category* category) {
        categories.erase(remove(categories.begin(), categories.end(), category), categories.end());
    }

    // Edit category
    void editCategory(Category* category, const string& name, const string& description) {
        category->setName(name);
        category->setDescription(description);
    }

    // Add product with category association
    void addProduct(Product* AssociationProduct) {
        products.push_back(AssociationProduct);
    }

    // Add order status
    void addOrderStatus(OrderStatus* status) {
        orderStatuses.push_back(status);
    }

    // Add order
    void addOrder(Order* order) {
        orders.push_back(order);
    }

    //generate products report
    void generateProductsReport() const {
        cout << "Товары в магазине: " << endl;
        for (const auto& product : products) {
            product->display();
        }
    }

    //generate customers report
    void generateCustomersReport() const {
        cout << "Покупатели в магазине: " << endl;
        for (const auto& customer : customers) {
            customer->display();
        }
    }

    //generate orders report
    void generateOrdersReport() const {
        cout << "Отчет по заказу: " << endl;
        for (const auto& order : orders) {
            order->display();
        }
    }

    // Generate category report
    void generateCategoriesReport() const {
        cout << "Categories in the store: " << endl;
        for (const auto& category : categories) {
            category->display();
        }
    }
};

int main() {
    OnlineStore store;
    Product product;

    // Create categories
    Category* electronics = new Category(1, "Electronics", "Devices and gadgets");
    Category* clothing = new Category(2, "Clothing", "Apparel and accessories");

    // Add categories to store
    store.addCategory(electronics);
    store.addCategory(clothing);

    // Create products and associate with categories
    Product* phone = new Product("Smartphone", "Latest model smartphone", 699.99, 10, electronics);
    Product* shirt = new Product("T-Shirt", "Comfortable cotton t-shirt", 19.99, 50, clothing);

    // Add products to store
    store.addProduct(phone);
    store.addProduct(shirt);

    // Generate reports
    store.generateCategoriesReport();
    store.generateProductsReport();

    // Cleanup
    delete electronics;
    delete clothing;
    delete phone;
    delete shirt;

    return 0;
}
