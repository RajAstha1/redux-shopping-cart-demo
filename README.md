# Redux Shopping Cart Demo

Redux state management demo for shopping cart - add, remove, and update items using Redux Toolkit

## 📋 Objective

This project demonstrates how to use **Redux** for state management in a shopping cart application. It showcases:

- ✅ **Add items** to cart
- ✅ **Remove items** from cart
- ✅ **Update quantity** of items
- ✅ **Calculate total price**
- ✅ **Clear cart**

This is a practical example using **Redux Toolkit** with React, making state management simple and efficient.

---

## 🚀 Setup Instructions

### Prerequisites

- Node.js (v14.0.0 or higher)
- npm or yarn package manager

### Installation Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/RajAstha1/redux-shopping-cart-demo.git
   cd redux-shopping-cart-demo
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```
   or
   ```bash
   yarn install
   ```

3. **Install Redux and Redux Toolkit** (if not already included)
   ```bash
   npm install @reduxjs/toolkit react-redux
   ```

4. **Start the development server**
   ```bash
   npm start
   ```
   or
   ```bash
   yarn start
   ```

5. **Open your browser** and navigate to `http://localhost:3000`

---

## 📁 Project Structure

```
redux-shopping-cart-demo/
├── src/
│   ├── components/
│   │   ├── Cart.jsx          # Shopping cart display component
│   │   ├── Product.jsx       # Individual product component
│   │   └── ProductList.jsx   # List of all products
│   ├── redux/
│   │   ├── slices/
│   │   │   └── cartSlice.js  # Cart slice with reducers and actions
│   │   └── store.js          # Redux store configuration
│   ├── App.jsx
│   └── index.js
├── public/
├── package.json
└── README.md
```

---

## 💻 Sample Code

### 1. Cart Slice (Redux Toolkit)

**File: `src/redux/slices/cartSlice.js`**

```javascript
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  items: [],
  totalQuantity: 0,
  totalPrice: 0,
};

const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    // Add item to cart
    addToCart(state, action) {
      const newItem = action.payload;
      const existingItem = state.items.find(item => item.id === newItem.id);

      if (existingItem) {
        // If item exists, increase quantity
        existingItem.quantity += newItem.quantity || 1;
      } else {
        // Add new item
        state.items.push({
          id: newItem.id,
          name: newItem.name,
          price: newItem.price,
          quantity: newItem.quantity || 1,
        });
      }
      state.totalQuantity += newItem.quantity || 1;
      state.totalPrice += newItem.price * (newItem.quantity || 1);
    },

    // Remove item from cart
    removeFromCart(state, action) {
      const id = action.payload;
      const existingItem = state.items.find(item => item.id === id);

      if (existingItem) {
        state.totalPrice -= existingItem.price * existingItem.quantity;
        state.totalQuantity -= existingItem.quantity;
        state.items = state.items.filter(item => item.id !== id);
      }
    },

    // Update item quantity
    updateQuantity(state, action) {
      const { id, quantity } = action.payload;
      const existingItem = state.items.find(item => item.id === id);

      if (existingItem) {
        const quantityDifference = quantity - existingItem.quantity;
        state.totalQuantity += quantityDifference;
        state.totalPrice += existingItem.price * quantityDifference;
        existingItem.quantity = quantity;
      }
    },

    // Clear entire cart
    clearCart(state) {
      state.items = [];
      state.totalQuantity = 0;
      state.totalPrice = 0;
    },
  },
});

export const cartActions = cartSlice.actions;
export default cartSlice.reducer;
```

### 2. Redux Store Configuration

**File: `src/redux/store.js`**

```javascript
import { configureStore } from '@reduxjs/toolkit';
import cartReducer from './slices/cartSlice';

const store = configureStore({
  reducer: {
    cart: cartReducer,
  },
});

export default store;
```

### 3. Shopping Cart Component

**File: `src/components/Cart.jsx`**

```javascript
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { cartActions } from '../redux/slices/cartSlice';
import './Cart.css';

const Cart = () => {
  const dispatch = useDispatch();
  const cartItems = useSelector(state => state.cart.items);
  const totalPrice = useSelector(state => state.cart.totalPrice);
  const totalQuantity = useSelector(state => state.cart.totalQuantity);

  const handleRemove = (id) => {
    dispatch(cartActions.removeFromCart(id));
  };

  const handleQuantityChange = (id, newQuantity) => {
    if (newQuantity > 0) {
      dispatch(cartActions.updateQuantity({ id, quantity: newQuantity }));
    }
  };

  const handleClearCart = () => {
    dispatch(cartActions.clearCart());
  };

  return (
    <div className="cart-container">
      <h1>Shopping Cart</h1>
      {cartItems.length === 0 ? (
        <p>Your cart is empty</p>
      ) : (
        <>
          <table className="cart-table">
            <thead>
              <tr>
                <th>Product</th>
                <th>Price</th>
                <th>Quantity</th>
                <th>Subtotal</th>
                <th>Action</th>
              </tr>
            </thead>
            <tbody>
              {cartItems.map(item => (
                <tr key={item.id}>
                  <td>{item.name}</td>
                  <td>${item.price.toFixed(2)}</td>
                  <td>
                    <input
                      type="number"
                      min="1"
                      value={item.quantity}
                      onChange={(e) => handleQuantityChange(item.id, parseInt(e.target.value))}
                    />
                  </td>
                  <td>${(item.price * item.quantity).toFixed(2)}</td>
                  <td>
                    <button onClick={() => handleRemove(item.id)}>Remove</button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
          <div className="cart-summary">
            <p><strong>Total Items: </strong>{totalQuantity}</p>
            <p><strong>Total Price: </strong>${totalPrice.toFixed(2)}</p>
          </div>
          <button className="clear-btn" onClick={handleClearCart}>Clear Cart</button>
        </>
      )}
    </div>
  );
};

export default Cart;
```

### 4. App Component

**File: `src/App.jsx`**

```javascript
import React, { useState } from 'react';
import { useDispatch } from 'react-redux';
import { cartActions } from './redux/slices/cartSlice';
import Cart from './components/Cart';
import './App.css';

const App = () => {
  const dispatch = useDispatch();
  const [products] = useState([
    { id: 1, name: 'Laptop', price: 999.99 },
    { id: 2, name: 'Mouse', price: 29.99 },
    { id: 3, name: 'Keyboard', price: 79.99 },
    { id: 4, name: 'Monitor', price: 299.99 },
  ]);

  const handleAddToCart = (product) => {
    dispatch(cartActions.addToCart(product));
    alert(`${product.name} added to cart!`);
  };

  return (
    <div className="app">
      <header>
        <h1>Redux Shopping Cart Demo</h1>
      </header>
      <main>
        <section className="products">
          <h2>Available Products</h2>
          <div className="product-grid">
            {products.map(product => (
              <div key={product.id} className="product-card">
                <h3>{product.name}</h3>
                <p>${product.price.toFixed(2)}</p>
                <button onClick={() => handleAddToCart(product)}>
                  Add to Cart
                </button>
              </div>
            ))}
          </div>
        </section>
        <section className="cart-section">
          <Cart />
        </section>
      </main>
    </div>
  );
};

export default App;
```

### 5. Main Entry Point

**File: `src/index.js`**

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import store from './redux/store';
import App from './App';
import './index.css';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

---

## ▶️ Run Instructions

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Start the development server:**
   ```bash
   npm start
   ```

3. **The app will open in your browser** at `http://localhost:3000`

4. **Test the functionality:**
   - Click "Add to Cart" to add products
   - View items in the shopping cart
   - Update quantities using the input field
   - Remove individual items
   - Clear entire cart

5. **Stop the server:**
   ```bash
   Press Ctrl + C
   ```

---

## 📚 Redux Concepts Explained

### 1. **Store**
The centralized state container that holds the entire application state. Created using `configureStore()` from Redux Toolkit.

### 2. **Slice**
A collection of Redux reducer logic and actions for a specific feature. In this project, we use `cartSlice` to manage cart state.

### 3. **Actions**
Plain objects that describe what happened. Example: `addToCart`, `removeFromCart`, `updateQuantity`, `clearCart`.

### 4. **Reducers**
Pure functions that take the current state and an action, and return a new state. Defined within slices.

### 5. **useSelector**
A React hook that allows components to read values from the Redux store.

```javascript
const items = useSelector(state => state.cart.items);
```

### 6. **useDispatch**
A React hook that returns the dispatch function to send actions to the Redux store.

```javascript
const dispatch = useDispatch();
dispatch(cartActions.addToCart(product));
```

### 7. **Redux Toolkit**
An official library that simplifies Redux development by providing utilities like `createSlice` and `configureStore`.

---

## 🔗 Documentation & Resources

- [Redux Official Documentation](https://redux.js.org/)
- [Redux Toolkit Guide](https://redux-toolkit.js.org/)
- [React-Redux Hooks Documentation](https://react-redux.js.org/api/hooks)
- [Redux DevTools Extension](https://github.com/reduxjs/redux-devtools-extension)
- [Redux Thunk for Async Actions](https://redux-toolkit.js.org/usage/usage-guide#async-thunks)

---

## 📦 Dependencies

- `react` - UI library
- `react-dom` - React DOM rendering
- `@reduxjs/toolkit` - Redux utilities and state management
- `react-redux` - React bindings for Redux

---

## 🎯 Future Enhancements

- Add product filtering and search
- Implement checkout functionality
- Add local storage persistence
- Create order history
- Add discount codes feature
- Implement product images
- Add to wishlist functionality

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

## 👨‍💻 Author

Created as a learning resource for Redux state management in React applications.

**Happy Coding! 🚀**
