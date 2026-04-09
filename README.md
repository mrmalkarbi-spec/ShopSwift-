# ShopSwift

A complete e-commerce application

## Folder Structure

- backend
  - src
  - tests
  - package.json
- frontend
  - src
  - public
  - package.json

backend/
├── package.json ✅
├── server.js ✅
├── .env.example ✅
├── models/
│   ├── User.js ✅
│   ├── Product.js ✅
│   ├── Order.js ✅
│   └── Review.js ✅
├── middleware/
│   └── auth.js ✅
└── routes/
    ├── auth.js ✅
    ├── products.js ✅
    ├── orders.js ✅
    ├── cart.js (بسويها)
    ├── users.js (بسويها)
    ├── admin.js (بسويها)
    └── reviews.js (بسويها)
{
  "name": "shopswift-backend",
  "version": "1.0.0",
  "description": "ShopSwift E-commerce Backend",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.0",
    "dotenv": "^16.0.3",
    "cors": "^2.8.5",
    "multer": "^1.4.5-lts.1",
    "nodemailer": "^6.9.1",
    "stripe": "^11.1.3",
    "axios": "^1.3.2"
  }
}
require('dotenv').config();
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ limit: '50mb', extended: true }));

// MongoDB Connection
mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/shopswift')
  .then(() => console.log('✅ MongoDB Connected'))
  .catch(err => console.log('❌ MongoDB Error:', err));

// Routes
app.use('/api/auth', require('./routes/auth'));
app.use('/api/products', require('./routes/products'));
app.use('/api/orders', require('./routes/orders'));
app.use('/api/cart', require('./routes/cart'));
app.use('/api/users', require('./routes/users'));
app.use('/api/admin', require('./routes/admin'));
app.use('/api/reviews', require('./routes/reviews'));

// Error Handling
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    message: err.message || 'Server Error'
  });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
});
MONGODB_URI=mongodb://localhost:27017/shopswift
JWT_SECRET=your_jwt_secret_key_change_this
NODE_ENV=development
PORT=5000

# Email (Resend)
RESEND_API_KEY=re_RLtddadC_D86oKQSDrXe6NsvNbLyUTnJ9
ADMIN_EMAIL=systembreach999@gmail.com

# Payment
STRIPE_SECRET_KEY=your_stripe_secret_key
PAYPAL_CLIENT_ID=AUV_hKpELHslPzAXtD195

# Shipping
SHIPPING_FEE_AED=10
SHIPPING_COUNTRY=UAE
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true
  },
  phone: {
    type: String,
    unique: true,
    sparse: true
  },
  password: {
    type: String,
    minlength: 6
  },
  firstName: String,
  lastName: String,
  profileImage: String,
  
  addresses: [{
    type: { type: String, enum: ['home', 'work', 'other'], default: 'home' },
    street: String,
    city: String,
    emirate: String,
    postalCode: String,
    isDefault: Boolean
  }],
  
  googleId: String,
  appleId: String,
  
  favoriteProducts: [mongoose.Schema.Types.ObjectId],
  preferredPaymentMethod: {
    type: String,
    enum: ['credit-card', 'apple-pay', 'paypal', 'bank-transfer']
  },
  language: {
    type: String,
    enum: ['ar', 'en'],
    default: 'ar'
  },
  
  isAdmin: { type: Boolean, default: false },
  isActive: { type: Boolean, default: true },
  isVerified: { type: Boolean, default: false },
  verificationToken: String,
  
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

// Hash password
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  
  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// Compare password
userSchema.methods.comparePassword = async function(passwordAttempt) {
  return await bcrypt.compare(passwordAttempt, this.password);
};

module.exports = mongoose.model('User', userSchema);
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
  name: {
    en: { type: String, required: true },
    ar: { type: String, required: true }
  },
  description: {
    en: String,
    ar: String
  },
  price: { type: Number, required: true },
  originalPrice: Number,
  discount: Number,
  
  images: [String],
  mainImage: String,
  
  category: {
    type: String,
    enum: ['electronics', 'clothing', 'books', 'home', 'sports', 'toys', 'beauty', 'automotive'],
    required: true
  },
  subcategory: String,
  
  color: [String],
  size: [String],
  specifications: {
    brand: String,
    model: String,
    warranty: String,
    material: String
  },
  
  stock: { type: Number, default: 0 },
  inStock: { type: Boolean, default: true },
  
  ratings: {
    average: { type: Number, default: 0, min: 0, max: 5 },
    count: { type: Number, default: 0 }
  },
  
  seller: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  
  isFeatured: Boolean,
  isHotDeal: Boolean,
  isNew: Boolean,
  isBestSeller: Boolean,
  
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

productSchema.index({ 'name.en': 'text', 'name.ar': 'text', 'description.en': 'text' });

module.exports = mongoose.model('Product', productSchema);
const mongoose = require('mongoose');

const orderSchema = new mongoose.Schema({
  orderNumber: { type: String, unique: true, required: true },
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  
  items: [{
    product: { type: mongoose.Schema.Types.ObjectId, ref: 'Product' },
    quantity: Number,
    price: Number,
    color: String,
    size: String
  }],
  
  subtotal: Number,
  shippingFee: { type: Number, default: 10 },
  tax: Number,
  discount: Number,
  total: Number,
  
  shippingAddress: {
    firstName: String,
    lastName: String,
    email: String,
    phone: String,
    street: String,
    city: String,
    emirate: String,
    postalCode: String,
    country: { type: String, default: 'UAE' }
  },
  
  status: {
    type: String,
    enum: ['pending', 'confirmed', 'processing', 'shipped', 'in-transit', 'delivered', 'cancelled'],
    default: 'pending'
  },
  
  trackingNumber: String,
  estimatedDelivery: Date,
  shippingCompany: String,
  
  paymentMethod: {
    type: String,
    enum: ['credit-card', 'apple-pay', 'paypal', 'bank-transfer'],
    required: true
  },
  paymentStatus: {
    type: String,
    enum: ['pending', 'completed', 'failed', 'refunded'],
    default: 'pending'
  },
  
  notes: String,
  cancelledAt: Date,
  cancellationReason: String,
  
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
  deliveredAt: Date
});

orderSchema.pre('save', async function(next) {
  if (!this.orderNumber) {
    const timestamp = Date.now().toString().slice(-8);
    this.orderNumber = `ORD-${timestamp}-${Math.random().toString(36).substr(2, 9).toUpperCase()}`;
  }
  next();
});

module.exports = mongoose.model('Order', orderSchema);
const mongoose = require('mongoose');

const reviewSchema = new mongoose.Schema({
  product: { type: mongoose.Schema.Types.ObjectId, ref: 'Product', required: true },
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  order: { type: mongoose.Schema.Types.ObjectId, ref: 'Order' },
  
  rating: { type: Number, required: true, min: 1, max: 5 },
  title: { en: String, ar: String },
  comment: { en: String, ar: String },
  
  helpful: { type: Number, default: 0 },
  isVerifiedPurchase: { type: Boolean, default: true },
  isApproved: { type: Boolean, default: true },
  
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Review', reviewSchema);
const jwt = require('jsonwebtoken');

const authMiddleware = (req, res, next) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) {
      return res.status(401).json({ message: 'No token provided' });
    }
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET || 'your_jwt_secret');
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ message: 'Invalid token' });
  }
};

const adminMiddleware = (req, res, next) => {
  if (!req.user?.isAdmin) {
    return res.status(403).json({ message: 'Admin access required' });
  }
  next();
};

module.exports = { authMiddleware, adminMiddleware };
const express = require('express');
const router = express.Router();
const User = require('../models/User');
const jwt = require('jsonwebtoken');
const { authMiddleware } = require('../middleware/auth');

// REGISTER
router.post('/register', async (req, res) => {
  try {
    const { email, password, firstName, lastName, phone } = req.body;
    
    if (!email || !password) {
      return res.status(400).json({ message: 'Email and password required' });
    }
    
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({ message: 'User already exists' });
    }
    
    const user = new User({ email, password, firstName, lastName, phone });
    await user.save();
    
    const token = jwt.sign(
      { id: user._id, email: user.email, isAdmin: user.isAdmin },
      process.env.JWT_SECRET || 'your_jwt_secret',
      { expiresIn: '7d' }
    );
    
    res.status(201).json({
      message: 'User registered successfully',
      token,
      user: { id: user._id, email: user.email, firstName: user.firstName }
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// LOGIN
router.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    if (!email || !password) {
      return res.status(400).json({ message: 'Email and password required' });
    }
    
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }
    
    const isPasswordValid = await user.comparePassword(password);
    if (!isPasswordValid) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }
    
    const token = jwt.sign(
      { id: user._id, email: user.email, isAdmin: user.isAdmin },
      process.env.JWT_SECRET || 'your_jwt_secret',
      { expiresIn: '7d' }
    );
    
    res.json({
      message: 'Login successful',
      token,
      user: { id: user._id, email: user.email, firstName: user.firstName, isAdmin: user.isAdmin }
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// LOGIN WITH PHONE
router.post('/login-phone', async (req, res) => {
  try {
    const { phone, password } = req.body;
    
    if (!phone || !password) {
      return res.status(400).json({ message: 'Phone and password required' });
    }
    
    const user = await User.findOne({ phone });
    if (!user) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }
    
    const isPasswordValid = await user.comparePassword(password);
    if (!isPasswordValid) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }
    
    const token = jwt.sign(
      { id: user._id, email: user.email, isAdmin: user.isAdmin },
      process.env.JWT_SECRET || 'your_jwt_secret',
      { expiresIn: '7d' }
    );
    
    res.json({
      message: 'Login successful',
      token,
      user: { id: user._id, phone: user.phone, firstName: user.firstName, isAdmin: user.isAdmin }
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GOOGLE LOGIN
router.post('/google', async (req, res) => {
  try {
    const { googleToken, email, firstName, lastName, profileImage } = req.body;
    
    let user = await User.findOne({ email });
    if (!user) {
      user = new User({
        email,
        firstName,
        lastName,
        profileImage,
        googleId: googleToken,
        isVerified: true
      });
      await user.save();
    }
    
    const token = jwt.sign(
      { id: user._id, email: user.email, isAdmin: user.isAdmin },
      process.env.JWT_SECRET || 'your_jwt_secret',
      { expiresIn: '7d' }
    );
    
    res.json({
      message: 'Google login successful',
      token,
      user: { id: user._id, email: user.email, firstName: user.firstName }
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// APPLE LOGIN
router.post('/apple', async (req, res) => {
  try {
    const { appleToken, email, firstName, lastName } = req.body;
    
    let user = await User.findOne({ email });
    if (!user) {
      user = new User({
        email,
        firstName,
        lastName,
        appleId: appleToken,
        isVerified: true
      });
      await user.save();
    }
    
    const token = jwt.sign(
      { id: user._id, email: user.email, isAdmin: user.isAdmin },
      process.env.JWT_SECRET || 'your_jwt_secret',
      { expiresIn: '7d' }
    );
    
    res.json({
      message: 'Apple login successful',
      token,
      user: { id: user._id, email: user.email, firstName: user.firstName }
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// VERIFY TOKEN
router.get('/verify', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    res.json({ user });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
const express = require('express');
const router = express.Router();
const Product = require('../models/Product');
const Review = require('../models/Review');
const { authMiddleware } = require('../middleware/auth');

// GET ALL PRODUCTS
router.get('/', async (req, res) => {
  try {
    const { category, minPrice, maxPrice, search, sort, page = 1, limit = 12 } = req.query;
    
    let filter = { inStock: true };
    
    if (category) filter.category = category;
    if (minPrice || maxPrice) {
      filter.price = {};
      if (minPrice) filter.price.$gte = Number(minPrice);
      if (maxPrice) filter.price.$lte = Number(maxPrice);
    }
    
    if (search) {
      filter.$or = [
        { 'name.en': { $regex: search, $options: 'i' } },
        { 'name.ar': { $regex: search, $options: 'i' } }
      ];
    }
    
    let sortOptions = {};
    if (sort === 'price-low') sortOptions.price = 1;
    if (sort === 'price-high') sortOptions.price = -1;
    if (sort === 'newest') sortOptions.createdAt = -1;
    if (sort === 'popular') sortOptions.isBestSeller = -1;
    
    const skip = (page - 1) * limit;
    
    const products = await Product.find(filter)
      .sort(sortOptions)
      .skip(skip)
      .limit(Number(limit));
    
    const total = await Product.countDocuments(filter);
    
    res.json({
      products,
      pagination: {
        current: Number(page),
        total: Math.ceil(total / limit),
        totalItems: total
      }
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET FEATURED PRODUCTS
router.get('/featured', async (req, res) => {
  try {
    const products = await Product.find({ isFeatured: true }).limit(10);
    res.json(products);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET BEST SELLERS
router.get('/bestsellers', async (req, res) => {
  try {
    const products = await Product.find({ isBestSeller: true }).limit(10);
    res.json(products);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET HOT DEALS
router.get('/hotdeals', async (req, res) => {
  try {
    const products = await Product.find({ isHotDeal: true }).limit(10);
    res.json(products);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET SINGLE PRODUCT
router.get('/:id', async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (!product) {
      return res.status(404).json({ message: 'Product not found' });
    }
    
    const reviews = await Review.find({ product: req.params.id }).populate('user', 'firstName lastName');
    
    res.json({ product, reviews });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// ADD REVIEW
router.post('/:id/review', authMiddleware, async (req, res) => {
  try {
    const { rating, title, comment } = req.body;
    
    const review = new Review({
      product: req.params.id,
      user: req.user.id,
      rating,
      title,
      comment
    });
    
    await review.save();
    
    const allReviews = await Review.find({ product: req.params.id });
    const avgRating = allReviews.reduce((sum, r) => sum + r.rating, 0) / allReviews.length;
    
    await Product.findByIdAndUpdate(req.params.id, {
      'ratings.average': avgRating,
      'ratings.count': allReviews.length
    });
    
    res.status(201).json(review);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET CATEGORIES
router.get('/categories/all', async (req, res) => {
  try {
    const categories = ['electronics', 'clothing', 'books', 'home', 'sports', 'toys', 'beauty', 'automotive'];
    res.json(categories);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
const express = require('express');
const router = express.Router();
const Order = require('../models/Order');
const User = require('../models/User');
const { authMiddleware } = require('../middleware/auth');

// CREATE ORDER
router.post('/', authMiddleware, async (req, res) => {
  try {
    const { items, shippingAddress, paymentMethod } = req.body;
    
    if (!items || items.length === 0) {
      return res.status(400).json({ message: 'Order must contain items' });
    }
    
    const subtotal = items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
    const shippingFee = 10;
    const tax = subtotal * 0.05;
    const total = subtotal + shippingFee + tax;
    
    const order = new Order({
      user: req.user.id,
      items,
      subtotal,
      shippingFee,
      tax,
      total,
      shippingAddress,
      paymentMethod,
      status: 'pending',
      estimatedDelivery: new Date(Date.now() + 3 * 24 * 60 * 60 * 1000)
    });
    
    await order.save();
    
    res.status(201).json({
      message: 'Order created successfully',
      order,
      estimatedDelivery: '3 days'
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET USER ORDERS
router.get('/user/my-orders', authMiddleware, async (req, res) => {
  try {
    const orders = await Order.find({ user: req.user.id })
      .populate('items.product')
      .sort({ createdAt: -1 });
    
    res.json(orders);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET ORDER DETAILS
router.get('/:orderId', authMiddleware, async (req, res) => {
  try {
    const order = await Order.findById(req.params.orderId).populate('items.product');
    
    if (!order) {
      return res.status(404).json({ message: 'Order not found' });
    }
    
    if (order.user.toString() !== req.user.id) {
      return res.status(403).json({ message: 'Not authorized' });
    }
    
    res.json(order);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// TRACK ORDER
router.get('/:orderId/track', async (req, res) => {
  try {
    const order = await Order.findById(req.params.orderId);
    
    if (!order) {
      return res.status(404).json({ message: 'Order not found' });
    }
    
    res.json({
      orderNumber: order.orderNumber,
      status: order.status,
      trackingNumber: order.trackingNumber,
      estimatedDelivery: order.estimatedDelivery,
      shippingCompany: order.shippingCompany || 'Standard Delivery',
      items: order.items,
      shippingAddress: order.shippingAddress
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// UPDATE ORDER STATUS (ADMIN)
router.patch('/:orderId/status', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    if (!user.isAdmin) {
      return res.status(403).json({ message: 'Admin access required' });
    }
    
    const { status, trackingNumber } = req.body;
    
    const order = await Order.findByIdAndUpdate(
      req.params.orderId,
      { status, trackingNumber, updatedAt: new Date() },
      { new: true }
    );
    
    res.json({ message: 'Order status updated', order });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// CANCEL ORDER
router.patch('/:orderId/cancel', authMiddleware, async (req, res) => {
  try {
    const { reason } = req.body;
    
    const order = await Order.findById(req.params.orderId);
    
    if (!order) {
      return res.status(404).json({ message: 'Order not found' });
    }
    
    if (order.user.toString() !== req.user.id) {
      return res.status(403).json({ message: 'Not authorized' });
    }
    
    if (['delivered', 'cancelled'].includes(order.status)) {
      return res.status(400).json({ message: 'Cannot cancel this order' });
    }
    
    order.status = 'cancelled';
    order.cancelledAt = new Date();
    order.cancellationReason = reason;
    
    await order.save();
    
    res.json({ message: 'Order cancelled successfully', order });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
const express = require('express');
const router = express.Router();
const Product = require('../models/Product');
const { authMiddleware } = require('../middleware/auth');

// ADD TO CART
router.post('/add', authMiddleware, async (req, res) => {
  try {
    const { productId, quantity, color, size } = req.body;
    
    const product = await Product.findById(productId);
    if (!product) {
      return res.status(404).json({ message: 'Product not found' });
    }
    
    if (quantity > product.stock) {
      return res.status(400).json({ message: 'Insufficient stock' });
    }
    
    res.json({
      message: 'Added to cart',
      item: {
        product: productId,
        quantity,
        color,
        size,
        price: product.price,
        name: product.name,
        image: product.mainImage
      }
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// REMOVE FROM CART
router.delete('/remove/:productId', authMiddleware, async (req, res) => {
  try {
    res.json({ message: 'Removed from cart' });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// UPDATE CART QUANTITY
router.patch('/update/:productId', authMiddleware, async (req, res) => {
  try {
    const { quantity } = req.body;
    
    res.json({ message: 'Cart updated', quantity });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET CART
router.get('/', authMiddleware, async (req, res) => {
  try {
    res.json({ items: [] });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
const express = require('express');
const router = express.Router();
const User = require('../models/User');
const Order = require('../models/Order');
const { authMiddleware } = require('../middleware/auth');

// GET PROFILE
router.get('/profile', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.user.id).select('-password');
    
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// UPDATE PROFILE
router.patch('/profile', authMiddleware, async (req, res) => {
  try {
    const { firstName, lastName, phone, profileImage } = req.body;
    
    const user = await User.findByIdAndUpdate(
      req.user.id,
      { firstName, lastName, phone, profileImage, updatedAt: new Date() },
      { new: true }
    ).select('-password');
    
    res.json({
      message: 'Profile updated successfully',
      user
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// ADD ADDRESS
router.post('/addresses', authMiddleware, async (req, res) => {
  try {
    const { type, street, city, emirate, postalCode, isDefault } = req.body;
    
    const user = await User.findById(req.user.id);
    
    if (isDefault) {
      user.addresses.forEach(addr => addr.isDefault = false);
    }
    
    user.addresses.push({
      type,
      street,
      city,
      emirate,
      postalCode,
      isDefault: isDefault || user.addresses.length === 0
    });
    
    await user.save();
    
    res.status(201).json({
      message: 'Address added',
      addresses: user.addresses
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET ADDRESSES
router.get('/addresses', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    res.json(user.addresses);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// UPDATE ADDRESS
router.patch('/addresses/:id', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    const address = user.addresses.id(req.params.id);
    
    if (!address) {
      return res.status(404).json({ message: 'Address not found' });
    }
    
    Object.assign(address, req.body);
    await user.save();
    
    res.json({ message: 'Address updated', address });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// DELETE ADDRESS
router.delete('/addresses/:id', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    user.addresses.id(req.params.id).remove();
    await user.save();
    
    res.json({ message: 'Address deleted' });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET FAVORITE PRODUCTS
router.get('/favorites', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.user.id).populate('favoriteProducts');
    res.json(user.favoriteProducts);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// ADD TO FAVORITES
router.post('/favorites/:productId', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    
    if (!user.favoriteProducts.includes(req.params.productId)) {
      user.favoriteProducts.push(req.params.productId);
      await user.save();
    }
    
    res.json({ message: 'Added to favorites' });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// REMOVE FROM FAVORITES
router.delete('/favorites/:productId', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    user.favoriteProducts = user.favoriteProducts.filter(id => id.toString() !== req.params.productId);
    await user.save();
    
    res.json({ message: 'Removed from favorites' });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// SET PREFERRED PAYMENT METHOD
router.patch('/payment-method', authMiddleware, async (req, res) => {
  try {
    const { paymentMethod } = req.body;
    
    const user = await User.findByIdAndUpdate(
      req.user.id,
      { preferredPaymentMethod: paymentMethod },
      { new: true }
    );
    
    res.json({ message: 'Payment method updated', paymentMethod: user.preferredPaymentMethod });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// SET LANGUAGE
router.patch('/language', authMiddleware, async (req, res) => {
  try {
    const { language } = req.body;
    
    const user = await User.findByIdAndUpdate(
      req.user.id,
      { language },
      { new: true }
    );
    
    res.json({ message: 'Language updated', language: user.language });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
const express = require('express');
const router = express.Router();
const Product = require('../models/Product');
const Order = require('../models/Order');
const User = require('../models/User');
const { authMiddleware } = require('../middleware/auth');

// Middleware: Check Admin
const checkAdmin = async (req, res, next) => {
  const user = await User.findById(req.user.id);
  if (!user.isAdmin) {
    return res.status(403).json({ message: 'Admin access required' });
  }
  next();
};

// ===== PRODUCTS =====

// CREATE PRODUCT
router.post('/products', authMiddleware, checkAdmin, async (req, res) => {
  try {
    const { name, description, price, category, images, mainImage, stock, color, size, specifications } = req.body;
    
    const product = new Product({
      name,
      description,
      price,
      category,
      images,
      mainImage,
      stock,
      color,
      size,
      specifications,
      seller: req.user.id,
      inStock: stock > 0
    });
    
    await product.save();
    
    res.status(201).json({
      message: 'Product created successfully',
      product
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// UPDATE PRODUCT
router.patch('/products/:id', authMiddleware, checkAdmin, async (req, res) => {
  try {
    const product = await Product.findByIdAndUpdate(
      req.params.id,
      { ...req.body, updatedAt: new Date() },
      { new: true }
    );
    
    res.json({
      message: 'Product updated successfully',
      product
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// DELETE PRODUCT
router.delete('/products/:id', authMiddleware, checkAdmin, async (req, res) => {
  try {
    await Product.findByIdAndDelete(req.params.id);
    
    res.json({ message: 'Product deleted successfully' });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET ALL PRODUCTS (ADMIN)
router.get('/products', authMiddleware, checkAdmin, async (req, res) => {
  try {
    const { page = 1, limit = 20, search } = req.query;
    
    let filter = {};
    if (search) {
      filter.$or = [
        { 'name.en': { $regex: search, $options: 'i' } },
        { 'name.ar': { $regex: search, $options: 'i' } }
      ];
    }
    
    const skip = (page - 1) * limit;
    
    const products = await Product.find(filter)
      .skip(skip)
      .limit(Number(limit));
    
    const total = await Product.countDocuments(filter);
    
    res.json({
      products,
      pagination: { current: Number(page), total: Math.ceil(total / limit) }
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// ===== ORDERS =====

// GET ALL ORDERS (ADMIN)
router.get('/orders', authMiddleware, checkAdmin, async (req, res) => {
  try {
    const { page = 1, limit = 20, status } = req.query;
    
    let filter = {};
    if (status) filter.status = status;
    
    const skip = (page - 1) * limit;
    
    const orders = await Order.find(filter)
      .populate('user', 'firstName lastName email phone')
      .populate('items.product')
      .skip(skip)
      .limit(Number(limit))
      .sort({ createdAt: -1 });
    
    const total = await Order.countDocuments(filter);
    
    res.json({
      orders,
      pagination: { current: Number(page), total: Math.ceil(total / limit) }
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET ORDER DETAILS (ADMIN)
router.get('/orders/:orderId', authMiddleware, checkAdmin, async (req, res) => {
  try {
    const order = await Order.findById(req.params.orderId)
      .populate('user')
      .populate('items.product');
    
    res.json(order);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// ===== USERS =====

// GET ALL USERS (ADMIN)
router.get('/users', authMiddleware, checkAdmin, async (req, res) => {
  try {
    const { page = 1, limit = 20, search } = req.query;
    
    let filter = {};
    if (search) {
      filter.$or = [
        { email: { $regex: search, $options: 'i' } },
        { firstName: { $regex: search, $options: 'i' } },
        { phone: { $regex: search, $options: 'i' } }
      ];
    }
    
    const skip = (page - 1) * limit;
    
    const users = await User.find(filter)
      .select('-password')
      .skip(skip)
      .limit(Number(limit));
    
    const total = await User.countDocuments(filter);
    
    res.json({
      users,
      pagination: { current: Number(page), total: Math.ceil(total / limit) }
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// ===== DASHBOARD STATISTICS =====

// GET STATISTICS
router.get('/stats/overview', authMiddleware, checkAdmin, async (req, res) => {
  try {
    const totalUsers = await User.countDocuments();
    const totalProducts = await Product.countDocuments();
    const totalOrders = await Order.countDocuments();
    const totalRevenue = await Order.aggregate([
      { $match: { paymentStatus: 'completed' } },
      { $group: { _id: null, total: { $sum: '$total' } } }
    ]);
    
    const recentOrders = await Order.find()
      .populate('user', 'firstName lastName')
      .sort({ createdAt: -1 })
      .limit(5);
    
    res.json({
      stats: {
        totalUsers,
        totalProducts,
        totalOrders,
        totalRevenue: totalRevenue[0]?.total || 0
      },
      recentOrders
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
const express = require('express');
const router = express.Router();
const Review = require('../models/Review');
const { authMiddleware } = require('../middleware/auth');

// GET PRODUCT REVIEWS
router.get('/product/:productId', async (req, res) => {
  try {
    const reviews = await Review.find({ product: req.params.productId })
      .populate('user', 'firstName lastName profileImage')
      .sort({ createdAt: -1 });
    
    res.json(reviews);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// CREATE REVIEW
router.post('/', authMiddleware, async (req, res) => {
  try {
    const { productId, rating, title, comment } = req.body;
    
    const review = new Review({
      product: productId,
      user: req.user.id,
      rating,
      title,
      comment
    });
    
    await review.save();
    
    res.status(201).json({
      message: 'Review created successfully',
      review
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// MARK REVIEW HELPFUL
router.patch('/:reviewId/helpful', async (req, res) => {
  try {
    const review = await Review.findByIdAndUpdate(
      req.params.reviewId,
      { $inc: { helpful: 1 } },
      { new: true }
    );
    
    res.json(review);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
const nodemailer = require('nodemailer');

// Configure your email service
const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASSWORD
  }
});

// Send Order Confirmation Email
const sendOrderConfirmationEmail = async (order, user) => {
  try {
    const mailOptions = {
      from: process.env.ADMIN_EMAIL,
      to: user.email,
      subject: `Order Confirmation - ${order.orderNumber}`,
      html: `
        <h2>Order Confirmed!</h2>
        <p>Thank you for your order, ${user.firstName}!</p>
        <p>Order Number: ${order.orderNumber}</p>
        <p>Total: AED ${order.total}</p>
        <p>Estimated Delivery: 3 Days</p>
        <p>You will receive updates about your order status.</p>
      `
    };
    
    await transporter.sendMail(mailOptions);
    console.log('✅ Order confirmation email sent');
  } catch (error) {
    console.log('❌ Email error:', error);
  }
};

// Send Order Status Update Email
const sendOrderStatusEmail = async (order, user, newStatus) => {
  try {
    const statusMessages = {
      confirmed: 'Your order has been confirmed',
      processing: 'We are preparing your order',
      shipped: 'Your order is on its way!',
      in-transit: 'Your order is being delivered',
      delivered: 'Your order has been delivered',
      cancelled: 'Your order has been cancelled'
    };
    
    const mailOptions = {
      from: process.env.ADMIN_EMAIL,
      to: user.email,
      subject: `Order Update - ${order.orderNumber}`,
      html: `
        <h2>Order Status Update</h2>
        <p>Hi ${user.firstName},</p>
        <p>${statusMessages[newStatus]}</p>
        <p>Order: ${order.orderNumber}</p>
        <p>Tracking Number: ${order.trackingNumber || 'N/A'}</p>
      `
    };
    
    await transporter.sendMail(mailOptions);
  } catch (error) {
    console.log('❌ Email error:', error);
  }
};

// Send Registration Welcome Email
const sendWelcomeEmail = async (user) => {
  try {
    const mailOptions = {
      from: process.env.ADMIN_EMAIL,
      to: user.email,
      subject: 'Welcome to ShopSwift!',
      html: `
        <h2>Welcome to ShopSwift!</h2>
        <p>Hi ${user.firstName},</p>
        <p>Thank you for creating an account with us.</p>
        <p>Start shopping now and enjoy great deals!</p>
      `
    };
    
    await transporter.sendMail(mailOptions);
  } catch (error) {
    console.log('❌ Email error:', error);
  }
};

module.exports = {
  sendOrderConfirmationEmail,
  sendOrderStatusEmail,
  sendWelcomeEmail
};
// Mock Payment Service (Ready for real integration)

const processPayment = async (paymentMethod, amount, paymentDetails) => {
  try {
    // Here you would integrate with actual payment gateway
    // For now, returning mock success
    
    console.log(`Processing ${paymentMethod} payment of AED ${amount}`);
    
    // Mock validation
    if (!amount || amount <= 0) {
      throw new Error('Invalid amount');
    }
    
    // In production, call Stripe or PayPal API
    return {
      success: true,
      transactionId: `TXN-${Date.now()}`,
      status: 'completed',
      amount,
      method: paymentMethod
    };
  } catch (error) {
    throw new Error(`Payment failed: ${error.message}`);
  }
};

module.exports = {
  processPayment
};
{
  "name": "shopswift-frontend",
  "version": "1.0.0",
  "description": "ShopSwift E-commerce Frontend",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.8.0",
    "axios": "^1.3.2",
    "zustand": "^4.3.2",
    "react-i18next": "^12.1.4",
    "lucide-react": "^0.263.1",
    "clsx": "^1.2.1"
  },
  "devDependencies": {
    "tailwindcss": "^3.2.7",
    "postcss": "^8.4.24",
    "autoprefixer": "^10.4.14"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "dev": "vite"
  }
}
import React, { useEffect } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { useStore } from './store/useStore';
import { I18nextProvider } from 'react-i18next';
import i18n from './i18n/i18n';

// Components
import Navbar from './components/Navbar';
import Footer from './components/Footer';

// Pages
import Home from './pages/Home';
import Products from './pages/Products';
import ProductDetail from './pages/ProductDetail';
import Cart from './pages/Cart';
import Checkout from './pages/Checkout';
import Orders from './pages/Orders';
import OrderTracking from './pages/OrderTracking';
import Profile from './pages/Profile';
import Login from './pages/Login';
import Register from './pages/Register';
import Admin from './pages/Admin';
import AdminProducts from './pages/AdminProducts';
import AdminOrders from './pages/AdminOrders';

function App() {
  const { checkAuth } = useStore();
  
  useEffect(() => {
    checkAuth();
  }, []);
  
  return (
    <I18nextProvider i18n={i18n}>
      <Router>
        <Navbar />
        <main className="min-h-screen">
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/products" element={<Products />} />
            <Route path="/product/:id" element={<ProductDetail />} />
            <Route path="/cart" element={<Cart />} />
            <Route path="/checkout" element={<Checkout />} />
            <Route path="/orders" element={<Orders />} />
            <Route path="/order/:id/track" element={<OrderTracking />} />
            <Route path="/profile" element={<Profile />} />
            <Route path="/login" element={<Login />} />
            <Route path="/register" element={<Register />} />
            <Route path="/admin" element={<Admin />} />
            <Route path="/admin/products" element={<AdminProducts />} />
            <Route path="/admin/orders" element={<AdminOrders />} />
          </Routes>
        </main>
        <Footer />
      </Router>
    </I18nextProvider>
  );
}

export default App;
import create from 'zustand';
import axios from 'axios';

const API = axios.create({
  baseURL: 'http://localhost:5000/api'
});

export const useStore = create((set, get) => ({
  user: null,
  token: localStorage.getItem('token'),
  cart: [],
  language: localStorage.getItem('language') || 'ar',
  
  // Auth
  checkAuth: async () => {
    const token = localStorage.getItem('token');
    if (token) {
      try {
        API.defaults.headers.common['Authorization'] = `Bearer ${token}`;
        const res = await API.get('/auth/verify');
        set({ user: res.data.user, token });
      } catch (error) {
        localStorage.removeItem('token');
      }
    }
  },
  
  login: async (email, password) => {
    try {
      const res = await API.post('/auth/login', { email, password });
      localStorage.setItem('token', res.data.token);
      API.defaults.headers.common['Authorization'] = `Bearer ${res.data.token}`;
      set({ user: res.data.user, token: res.data.token });
      return res.data;
    } catch (error) {
      throw error.response?.data || error;
    }
  },
  
  register: async (email, password, firstName, lastName) => {
    try {
      const res = await API.post('/auth/register', {
        email,
        password,
        firstName,
        lastName
      });
      localStorage.setItem('token', res.data.token);
      API.defaults.headers.common['Authorization'] = `Bearer ${res.data.token}`;
      set({ user: res.data.user, token: res.data.token });
      return res.data;
    } catch (error) {
      throw error.response?.data || error;
    }
  },
  
  logout: () => {
    localStorage.removeItem('token');
    set({ user: null, token: null, cart: [] });
  },
  
  // Cart
  addToCart: (item) => {
    set((state) => {
      const existing = state.cart.find(c => c.id === item.id);
      if (existing) {
        return {
          cart: state.cart.map(c =>
            c.id === item.id ? { ...c, quantity: c.quantity + item.quantity } : c
          )
        };
      }
      return { cart: [...state.cart, item] };
    });
  },
  
  removeFromCart: (itemId) => {
    set((state) => ({
      cart: state.cart.filter(c => c.id !== itemId)
    }));
  },
  
  updateCart: (itemId, quantity) => {
    set((state) => ({
      cart: state.cart.map(c =>
        c.id === itemId ? { ...c, quantity } : c
      ).filter(c => c.quantity > 0)
    }));
  },
  
  clearCart: () => set({ cart: [] }),
  
  // Language
  setLanguage: (lang) => {
    localStorage.setItem('language', lang);
    set({ language: lang });
  }
}));
import React, { useState } from 'react';
import { Link } from 'react-router-dom';
import { useStore } from '../store/useStore';
import { useTranslation } from 'react-i18next';
import { ShoppingCart, Menu, X, LogOut, User } from 'lucide-react';

export default function Navbar() {
  const { user, logout, cart, setLanguage, language } = useStore();
  const { t, i18n } = useTranslation();
  const [isOpen, setIsOpen] = useState(false);
  
  const toggleLanguage = () => {
    const newLang = language === 'ar' ? 'en' : 'ar';
    setLanguage(newLang);
    i18n.changeLanguage(newLang);
  };
  
  return (
    <nav className="bg-gradient-to-r from-purple-600 to-pink-600 text-white sticky top-0 z-50">
      <div className="container mx-auto px-4 py-3">
        <div className="flex justify-between items-center">
          {/* Logo */}
          <Link to="/" className="text-2xl font-bold">ShopSwift</Link>
          
          {/* Desktop Menu */}
          <div className="hidden md:flex gap-8 items-center">
            <Link to="/products" className="hover:text-gray-200">{t('products')}</Link>
            <Link to="/orders" className="hover:text-gray-200">{t('orders')}</Link>
            
            {user?.isAdmin && (
              <Link to="/admin" className="hover:text-gray-200">{t('admin')}</Link>
            )}
            
            <button onClick={toggleLanguage} className="px-3 py-1 bg-white text-purple-600 rounded">
              {language === 'ar' ? 'EN' : 'AR'}
            </button>
            
            <Link to="/cart" className="relative">
              <ShoppingCart size={24} />
              {cart.length > 0 && (
                <span className="absolute -top-2 -right-2 bg-red-500 text-white rounded-full w-5 h-5 flex items-center justify-center text-xs">
                  {cart.length}
                </span>
              )}
            </Link>
            
            {user ? (
              <div className="flex gap-3 items-center">
                <Link to="/profile" className="hover:text-gray-200 flex items-center gap-1">
                  <User size={20} /> {user.firstName}
                </Link>
                <button onClick={logout} className="hover:text-gray-200">
                  <LogOut size={20} />
                </button>
              </div>
            ) : (
              <div className="flex gap-3">
                <Link to="/login" className="px-4 py-2 bg-white text-purple-600 rounded font-bold">
                  {t('login')}
                </Link>
                <Link to="/register" className="px-4 py-2 border-2 border-white rounded font-bold">
                  {t('register')}
                </Link>
              </div>
            )}
          </div>
          
          {/* Mobile Menu Button */}
          <button className="md:hidden" onClick={() => setIsOpen(!isOpen)}>
            {isOpen ? <X size={24} /> : <Menu size={24} />}
          </button>
        </div>
        
        {/* Mobile Menu */}
        {isOpen && (
          <div className="md:hidden mt-4 flex flex-col gap-4">
            <Link to="/products">{t('products')}</Link>
            <Link to="/orders">{t('orders')}</Link>
            <Link to="/cart" className="flex items-center gap-2">
              <ShoppingCart size={20} /> {t('cart')}
            </Link>
            <button onClick={toggleLanguage} className="text-left">
              {language === 'ar' ? 'English' : 'العربية'}
            </button>
            
            {!user && (
              <div className="flex flex-col gap-2">
                <Link to="/login" className="px-4 py-2 bg-white text-purple-600 rounded text-center">
                  {t('login')}
                </Link>
              </div>
            )}
          </div>
        )}
      </div>
    </nav>
  );
}
import React from 'react';
import { Link } from 'react-router-dom';
import { Star, ShoppingCart, Heart } from 'lucide-react';
import { useStore } from '../store/useStore';
import { useTranslation } from 'react-i18next';

export default function ProductCard({ product }) {
  const { addToCart } = useStore();
  const { t } = useTranslation();
  
  const handleAddToCart = () => {
    addToCart({
      id: product._id,
      name: product.name,
      price: product.price,
      image: product.mainImage,
      quantity: 1
    });
  };
  
  return (
    <div className="bg-white rounded-lg shadow-md hover:shadow-lg transition-all overflow-hidden">
      {/* Image */}
      <Link to={`/product/${product._id}`} className="relative overflow-hidden bg-gray-200 h-64">
        <img 
          src={product.mainImage} 
          alt={product.name.en}
          className="w-full h-full object-cover hover:scale-110 transition-transform duration-300"
        />
        {product.discount && (
          <div className="absolute top-3 right-3 bg-red-500 text-white px-3 py-1 rounded text-sm font-bold">
            -{product.discount}%
          </div>
        )}
      </Link>
      
      {/* Details */}
      <div className="p-4">
        <Link to={`/product/${product._id}`} className="hover:text-purple-600">
          <h3 className="font-bold text-lg truncate">{product.name.en}</h3>
        </Link>
        
        {/* Rating */}
        <div className="flex items-center gap-1 mt-2">
          {[...Array(5)].map((_, i) => (
            <Star
              key={i}
              size={16}
              className={i < Math.round(product.ratings.average) ? 'fill-yellow-400 text-yellow-400' : 'text-gray-300'}
            />
          ))}
          <span className="text-sm text-gray-600">({product.ratings.count})</span>
        </div>
        
        {/* Price */}
        <div className="flex items-center gap-2 mt-3">
          <span className="text-xl font-bold text-purple-600">AED {product.price}</span>
          {product.originalPrice && (
            <span className="text-sm text-gray-500 line-through">AED {product.originalPrice}</span>
          )}
        </div>
        
        {/* Actions */}
        <div className="flex gap-2 mt-4">
          <button
            onClick={handleAddToCart}
            className="flex-1 bg-purple-600 text-white py-2 rounded hover:bg-purple-700 flex items-center justify-center gap-2"
          >
            <ShoppingCart size={18} /> {t('addCart')}
          </button>
          <button className="flex-1 border-2 border-purple-600 text-purple-600 py-2 rounded hover:bg-purple-50">
            <Heart size={18} className="mx-auto" />
          </button>
        </div>
      </div>
    </div>
  );
}
import React, { useEffect, useState } from 'react';
import { useTranslation } from 'react-i18next';
import axios from 'axios';
import ProductCard from '../components/ProductCard';
import { ChevronRight } from 'lucide-react';

const API = axios.create({
  baseURL: 'http://localhost:5000/api'
});

export default function Home() {
  const { t } = useTranslation();
  const [featuredProducts, setFeaturedProducts] = useState([]);
  const [bestSellers, setBestSellers] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchProducts();
  }, []);
  
  const fetchProducts = async () => {
    try {
      const [featured, sellers] = await Promise.all([
        API.get('/products/featured'),
        API.get('/products/bestsellers')
      ]);
      
      setFeaturedProducts(featured.data);
      setBestSellers(sellers.data);
    } catch (error) {
      console.error('Error fetching products:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="min-h-screen bg-gray-50">
      {/* Hero Banner */}
      <section className="bg-gradient-to-r from-purple-600 to-pink-600 text-white py-16">
        <div className="container mx-auto px-4 text-center">
          <h1 className="text-5xl font-bold mb-4">{t('welcome')}</h1>
          <p className="text-xl mb-8">Discover amazing products at great prices</p>
          <button className="bg-white text-purple-600 px-8 py-3 rounded-lg font-bold hover:bg-gray-100">
            {t('shopNow')}
          </button>
        </div>
      </section>
      
      {/* Featured Products */}
      <section className="py-12">
        <div className="container mx-auto px-4">
          <div className="flex justify-between items-center mb-8">
            <h2 className="text-3xl font-bold">{t('featured')}</h2>
            <a href="/products" className="text-purple-600 font-bold flex items-center gap-2 hover:gap-3">
              View All <ChevronRight size={20} />
            </a>
          </div>
          
          {loading ? (
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
              {[...Array(4)].map((_, i) => (
                <div key={i} className="bg-gray-200 rounded-lg h-80 animate-pulse"></div>
              ))}
            </div>
          ) : (
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
              {featuredProducts.map(product => (
                <ProductCard key={product._id} product={product} />
              ))}
            </div>
          )}
        </div>
      </section>
      
      {/* Best Sellers */}
      <section className="py-12 bg-white">
        <div className="container mx-auto px-4">
          <div className="flex justify-between items-center mb-8">
            <h2 className="text-3xl font-bold">{t('bestSellers')}</h2>
          </div>
          
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
            {bestSellers.map(product => (
              <ProductCard key={product._id} product={product} />
            ))}
          </div>
        </div>
      </section>
    </div>
  );
}
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import ar from './ar.json';
import en from './en.json';

i18n
  .use(initReactI18next)
  .init({
    resources: { ar: { translation: ar }, en: { translation: en } },
    lng: localStorage.getItem('language') || 'ar',
    interpolation: { escapeValue: false }
  });

export default i18n;
{
  "welcome": "مرحباً بك في ShopSwift",
  "shopNow": "ابدأ التسوق",
  "products": "المنتجات",
  "orders": "الطلبات",
  "admin": "لوحة الإدارة",
  "login": "دخول",
  "register": "إنشاء حساب",
  "cart": "السلة",
  "addCart": "أضف إلى السلة",
  "featured": "المنتجات المختارة",
  "bestSellers": "الأكثر مبيعاً",
  "price": "السعر",
  "quantity": "الكمية",
  "total": "الإجمالي",
  "checkout": "الدفع",
  "shipping": "الشحن",
  "payment": "الدفع",
  "profile": "الملف الشخصي",
  "logout": "تسجيل الخروج",
  "favorite": "المفضلة",
  "rating": "التقييم",
  "review": "التعليق"
}
{
  "welcome": "Welcome to ShopSwift",
  "shopNow": "Start Shopping",
  "products": "Products",
  "orders": "Orders",
  "admin": "Admin Panel",
  "login": "Login",
  "register": "Register",
  "cart": "Cart",
  "addCart": "Add to Cart",
  "featured": "Featured Products",
  "bestSellers": "Best Sellers",
  "price": "Price",
  "quantity": "Quantity",
  "total": "Total",
  "checkout": "Checkout",
  "shipping": "Shipping",
  "payment": "Payment",
  "profile": "Profile",
  "logout": "Logout",
  "favorite": "Favorites",
  "rating": "Rating",
  "review": "Review"
}
import React from 'react';
import { Link } from 'react-router-dom';
import { useStore } from '../store/useStore';
import { useTranslation } from 'react-i18next';
import { Trash2, Plus, Minus } from 'lucide-react';

export default function Cart() {
  const { cart, removeFromCart, updateCart, clearCart } = useStore();
  const { t } = useTranslation();
  
  const total = cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  const shipping = 10;
  const tax = total * 0.05;
  const grandTotal = total + shipping + tax;
  
  if (cart.length === 0) {
    return (
      <div className="container mx-auto px-4 py-16 text-center">
        <h1 className="text-3xl font-bold mb-4">{t('cart')}</h1>
        <p className="text-gray-600 mb-8">Your cart is empty</p>
        <Link to="/products" className="bg-purple-600 text-white px-8 py-3 rounded-lg inline-block">
          Continue Shopping
        </Link>
      </div>
    );
  }
  
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">{t('cart')}</h1>
      
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
        {/* Cart Items */}
        <div className="lg:col-span-2 space-y-4">
          {cart.map(item => (
            <div key={item.id} className="flex gap-4 bg-white p-4 rounded-lg shadow">
              <img src={item.image} alt={item.name} className="w-24 h-24 object-cover rounded" />
              
              <div className="flex-1">
                <h3 className="font-bold text-lg">{item.name}</h3>
                <p className="text-purple-600 text-lg font-bold">AED {item.price}</p>
              </div>
              
              <div className="flex items-center gap-2">
                <button onClick={() => updateCart(item.id, item.quantity - 1)} className="p-2 hover:bg-gray-100">
                  <Minus size={18} />
                </button>
                <span className="w-8 text-center font-bold">{item.quantity}</span>
                <button onClick={() => updateCart(item.id, item.quantity + 1)} className="p-2 hover:bg-gray-100">
                  <Plus size={18} />
                </button>
              </div>
              
              <button onClick={() => removeFromCart(item.id)} className="text-red-500 hover:text-red-700">
                <Trash2 size={20} />
              </button>
            </div>
          ))}
        </div>
        
        {/* Summary */}
        <div className="bg-white rounded-lg shadow p-6 h-fit">
          <h2 className="text-2xl font-bold mb-4">Order Summary</h2>
          
          <div className="space-y-3 mb-4 border-b pb-4">
            <div className="flex justify-between">
              <span>Subtotal:</span>
              <span>AED {total.toFixed(2)}</span>
            </div>
            <div className="flex justify-between">
              <span>Shipping:</span>
              <span>AED {shipping}</span>
            </div>
            <div className="flex justify-between">
              <span>Tax (5%):</span>
              <span>AED {tax.toFixed(2)}</span>
            </div>
          </div>
          
          <div className="flex justify-between text-xl font-bold mb-6">
            <span>Total:</span>
            <span>AED {grandTotal.toFixed(2)}</span>
          </div>
          
          <Link to="/checkout" className="w-full bg-purple-600 text-white py-3 rounded-lg font-bold text-center hover:bg-purple-700 block mb-3">
            {t('checkout')}
          </Link>
          
          <button onClick={() => clearCart()} className="w-full border-2 border-gray-300 py-3 rounded-lg font-bold hover:bg-gray-50">
            Clear Cart
          </button>
        </div>
      </div>
    </div>
  );
}
# PayPal
PAYPAL_CLIENT_ID=AUV_hKpELHslPzAXtD195
PAYPAL_CLIENT_SECRET=AUV_hKpELHslPzAXtD195RqyIdDAgRLI0HrvnSIU8oJx6ByK5uCFVWS4LtjMGkyMfPjaTIpMJFQOlrZG

# Email
RESEND_API_KEY=re_RLtddadC_D86oKQSDrXe6NsvNbLyUTnJ9

# Stripe (مطلوب لـ Apple Pay)
STRIPE_SECRET_KEY=sk_test_...
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

// إنشاء Payment Intent يدعم Apple Pay والسحب التلقائي
router.post('/create-payment-intent', authMiddleware, async (req, res) => {
  try {
    const { amount, currency = 'aed', customerId } = req.body;

    const paymentIntent = await stripe.paymentIntents.create({
      amount: amount * 100, // السعر بالفلس
      currency: currency,
      customer: customerId, // للسحب التلقائي مستقبلاً
      setup_future_usage: 'off_session', // يسمح بالسحب لاحقاً دون تواجد المستخدم
      payment_method_types: ['card', 'apple_pay'], 
    });

    res.json({ clientSecret: paymentIntent.client_secret });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
const { Resend } = require('resend');
const resend = new Resend(process.env.RESEND_API_KEY);

const sendOrderEmail = async (userEmail, orderDetails) => {
  try {
    await resend.emails.send({
      from: 'ShopSwift <onboarding@resend.dev>', // أو دومينك الخاص
      to: userEmail,
      subject: `تأكيد الطلب #${orderDetails.orderNumber}`,
      html: `<h1>شكراً لتسوقك معنا!</h1><p>إجمالي طلبك هو: ${orderDetails.total} درهم</p>`
    });
    console.log('✅ Email sent successfully');
  } catch (error) {
    console.error('❌ Email failed:', error);
  }
};
