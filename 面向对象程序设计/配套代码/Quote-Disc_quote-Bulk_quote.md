```c++
#include<iostream>
#include<string>
using namespace std;

class Quote {
public:
	Quote() = default;
	Quote(const string &book, double sales_price):bookNo(book), price(sales_price){ }
	string isbn() const { return bookNo; }  // 返回书籍的ISBN编号
	virtual double net_price(size_t n) const { return n * price; }  // 返回书籍的时机销售价格
	virtual ~Quote() = default;
private:
	string bookNo;
protected:
	double price = 0.0;
};

class Disc_quote : public  Quote{
public:
	Disc_quote() = default;
	Disc_quote(const string& book, double price, size_t qty, double disc) : 
				Quote(book, price),
				quantity(qty), discount(disc){ }
	double net_price(size_t) const = 0;
	pair<size_t, double> discount_policy() const {
		return { quantity, discount };
	}
protected:
	size_t quantity = 0;
	double discount = 0.0;
};

class Bulk_quote : public Disc_quote {
public:
	Bulk_quote() = default;
	Bulk_quote(const string&, double, size_t, double);
	double net_price(size_t) const override;
};
Bulk_quote::Bulk_quote(const string& book, double price, size_t qty, double disc): Disc_quote(book, price, qty, disc){ }
double Bulk_quote::net_price(size_t cnt) const {
	if (cnt >= quantity)
		return cnt * discount * price;
	else
		return cnt * price;
}
double print_total(ostream& os, const Quote &item, size_t n) {
	double ret = item.net_price(n);
	os << "ISBN：" << item.isbn() << " # sold：" << n << " total due：" << ret << endl;
	return ret;
}
```

