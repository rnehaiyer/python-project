# python-project
rom flask import Flask, render_template, request, redirect, url_for, flash
import sqlite3

app = Flask(__name__)
app.secret_key = 'your_secret_key'  # Change this to a random secret key

# Database functions
def init_db():
    with sqlite3.connect('database.db') as conn:
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS products (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                price REAL NOT NULL
            )
        ''')
        conn.commit()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/add_product', methods=['GET', 'POST'])
def add_product():
    if request.method == 'POST':
        name = request.form['name']
        price = request.form['price']
        with sqlite3.connect('database.db') as conn:
            cursor = conn.cursor()
            cursor.execute('INSERT INTO products (name, price) VALUES (?, ?)', (name, price))
            conn.commit()
        flash('Product added successfully!')
        return redirect(url_for('index'))
    return render_template('add_product.html')

@app.route('/view_products')
def view_products():
    with sqlite3.connect('database.db') as conn:
        cursor = conn.cursor()
        cursor.execute('SELECT * FROM products')
        products = cursor.fetchall()
    return render_template('view_products.html', products=products)

@app.route('/billing', methods=['GET', 'POST'])
def billing():
    if request.method == 'POST':
        product_ids = request.form.getlist('product_ids')
        quantities = request.form.getlist('quantities')
        total_amount = 0
        purchased_items = []

        for product_id, quantity in zip(product_ids, quantities):
            if quantity.isdigit() and int(quantity) > 0:
                quantity = int(quantity)
                with sqlite3.connect('database.db') as conn:
                    cursor = conn.cursor()
                    cursor.execute('SELECT name, price FROM products WHERE id = ?', (product_id,))
                    product = cursor.fetchone()
                    if product:
                        item_name = product[0]
                        item_price = product[1]
                        total_amount += item_price * quantity
                        purchased_items.append((item_name, item_price, quantity))

        return render_template('billing.html', total_amount=total_amount, purchased_items=purchased_items)

    with sqlite3.connect('database.db') as conn:
        cursor = conn.cursor()
        cursor.execute('SELECT * FROM products')
        products = cursor.fetchall()
    return render_template('billing.html', products=products)

if __name__ == '__main__':
    init_db()  # Initialize the database
    app.run(debug=True)
