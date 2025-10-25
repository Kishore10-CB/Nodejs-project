# Nodejs-project
{
  "name": "ecommerce-cart",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2"
  }
}

const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');

const app = express();
app.use(cors());
app.use(bodyParser.json());
app.use(express.static('public'));

let products = [
  { id: 1, name: "Laptop", price: 999 },
  { id: 2, name: "Headphones", price: 199 },
  { id: 3, name: "Mouse", price: 49 }
];

let cart = [];

app.get('/api/products', (req, res) => {
  res.json(products);
});

app.get('/api/cart', (req, res) => {
  res.json(cart);
});

app.post('/api/cart', (req, res) => {
  const { productId, quantity } = req.body;
  const product = products.find(p => p.id === productId);
  if (!product) return res.status(404).json({ message: "Product not found" });

  const existing = cart.find(item => item.productId === productId);
  if (existing) {
    existing.quantity += quantity;
  } else {
    cart.push({ productId, quantity });
  }

  res.json(cart);
});

app.delete('/api/cart/:id', (req, res) => {
  const id = parseInt(req.params.id);
  cart = cart.filter(item => item.productId !== id);
  res.json(cart);
});

app.listen(3000, () => console.log('Server running on http://localhost:3000'));

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>E-commerce Cart</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h1>🛍️ My Store</h1>
  <div id="products"></div>
  <h2>Your Cart</h2>
  <ul id="cart"></ul>

  <script src="script.js"></script>
</body>
</html>

async function loadProducts() {
  const res = await fetch('/api/products');
  const products = await res.json();
  const container = document.getElementById('products');
  container.innerHTML = products.map(p => `
    <div>
      <strong>${p.name}</strong> - $${p.price}
      <button onclick="addToCart(${p.id})">Add to Cart</button>
    </div>
  `).join('');
}

async function loadCart() {
  const res = await fetch('/api/cart');
  const cart = await res.json();
  const resProducts = await fetch('/api/products');
  const products = await resProducts.json();

  document.getElementById('cart').innerHTML = cart.map(item => {
    const product = products.find(p => p.id === item.productId);
    return `<li>${product.name} x ${item.quantity} 
      <button onclick="removeFromCart(${item.productId})">Remove</button></li>`;
  }).join('');
}

async function addToCart(id) {
  await fetch('/api/cart', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ productId: id, quantity: 1 })
  });
  loadCart();
}

async function removeFromCart(id) {
  await fetch(`/api/cart/${id}`, { method: 'DELETE' });
  loadCart();
}

loadProducts();
loadCart();
