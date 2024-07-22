##後端部分
**建立資料庫連接與路由
```javascript
const express = require('express'); // 引入Express框架
const router = express.Router(); // 建立一個新的路由器實例
const mysql = require('mysql'); // 引入MySQL模組

// 建立資料庫連接設定
const db = mysql.createConnection({
  host: '127.0.0.1', // 資料庫主機位址
  user: 'root', // 資料庫使用者名稱
  password: '123456', // 資料庫使用者密碼
  database: 'test' // 資料庫名稱
});

// 連接資料庫
db.connect(err => {
  if (err) {
    console.error('Database connection failed: ' + err.stack); // 連接失敗時打印錯誤
    return;
  }
  console.log('Connected to database.'); // 連接成功時打印訊息
});

// 定義一個GET路由來獲取產品資料
router.get('/products', (req, res) => {
  // 從資料庫中選取所有產品資料
  db.query('SELECT * FROM products', (err, results) => {
    if (err) {
      res.status(500).send('Database query failed'); // 查詢失敗時返回錯誤訊息
      return;
    }
    res.json(results); // 查詢成功時返回結果
  });
});

module.exports = router; // 匯出路由模組
```
**在伺服器中使用這個路由
```javascript
const express = require('express'); // 引入Express框架
const app = express(); // 建立一個Express應用程式實例
const productsRouter = require('./productsRouter'); // 引入剛剛建立的產品路由

app.use('/api', productsRouter); // 使用產品路由，前綴為/api

app.listen(3005, () => {
  console.log('Server is running on port 3005'); // 啟動伺服器並監聽3005端口
});
```
##前端部分
**向後端fetch資料並設置KEY值，使用map整理陣列
```javascript
import React, { useState, useEffect } from 'react';

const ProductList = () => {
  const [products, setProducts] = useState([]); // 定義狀態來保存產品資料

  useEffect(() => {
    // 使用fetch API從後端獲取產品資料
    fetch('/api/products')
      .then(response => response.json()) // 將回應轉換為JSON
      .then(data => setProducts(data)) // 設定產品資料到狀態中
      .catch(error => console.error('Error fetching products:', error)); // 處理錯誤
  }, []); // 空陣列確保這個副作用只會在組件掛載時執行一次

  return (
    <div>
      {products.map(product => (
        <div key={product.id}> {/* 使用產品ID作為每個產品項目的鍵 */}
          <h2>{product.name}</h2> {/* 顯示產品名稱 */}
          <p>{product.description}</p> {/* 顯示產品描述 */}
          <p>{product.price}</p> {/* 顯示產品價格 */}
        </div>
      ))}
    </div>
  );
};

export default ProductList; // 匯出產品列表組件
```
**使用Context來保存購物車資料
```javascript
import React, { createContext, useContext, useState } from 'react';

// 建立一個新的Context來保存購物車資料
const CartContext = createContext();

export const useCart = () => {
  return useContext(CartContext); // 使用useContext鉤子來獲取購物車Context
};

// 建立一個提供者組件來包裹應用程式
export const CartProvider = ({ children }) => {
  const [cart, setCart] = useState([]); // 定義狀態來保存購物車資料

  // 定義一個函數來添加產品到購物車
  const addToCart = (product) => {
    setCart([...cart, product]); // 將新的產品添加到購物車
  };

  // 定義一個函數來清空購物車
  const clearCart = () => {
    setCart([]); // 清空購物車
  };

  return (
    <CartContext.Provider value={{ cart, addToCart, clearCart }}> {/* 提供購物車資料與操作函數 */}
      {children} {/* 渲染子組件 */}
    </CartContext.Provider>
  );
};
```
**結帳時，使用第三方金流（例如LinePay）的createOrder路由
```javascript
import React from 'react';
import { useCart } from './CartContext'; // 引入自定義的useCart鉤子

const Checkout = () => {
  const { cart, clearCart } = useCart(); // 獲取購物車資料與操作函數

  // 定義處理結帳的函數
  const handleCheckout = () => {
    // 發送購物車資料到後端的createOrder路由
    fetch('/api/createOrder', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json' // 設定內容類型為JSON
      },
      body: JSON.stringify({ cart }) // 將購物車資料轉換為JSON字串
    })
    .then(response => response.json()) // 將回應轉換為JSON
    .then(data => {
      if (data.success) {
        window.location.href = data.paymentUrl; // 成功時跳轉到支付頁面
      } else {
        console.error('Checkout failed:', data.message); // 失敗時打印錯誤訊息
      }
    })
    .catch(error => console.error('Error during checkout:', error)); // 處理錯誤
  };

  return (
    <button onClick={handleCheckout}>Checkout</button> // 結帳按鈕
  );
};

export default Checkout; // 匯出結帳組件
```
