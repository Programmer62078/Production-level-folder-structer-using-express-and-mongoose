# Production-level-folder-structer-using-express-and-mongoose
production level folder structure 



To set up the folder structure and code for a production-grade Node.js backend, I'll walk you through creating basic files and structure for each directory. The following steps assume you're using **Express.js** as the web framework and **MongoDB** with **Mongoose** as the database.

### Step 1: Initialize the Project
First, initialize the project using `npm`:
```bash
mkdir node-backend
cd node-backend
npm init -y
npm install express mongoose dotenv winston
npm install --save-dev nodemon jest supertest
```

### Step 2: Folder Structure Setup

Now, create the folder structure:

```bash
mkdir -p src/config src/controllers src/models src/routes src/middlewares src/services src/utils src/loaders src/jobs src/tests
mkdir logs
touch src/app.js src/server.js src/config/{db.js,env.js,index.js} src/controllers/{authController.js,userController.js,index.js} src/models/{userModel.js,index.js} src/routes/{authRoutes.js,userRoutes.js,index.js} src/middlewares/{authMiddleware.js,errorMiddleware.js,index.js} src/services/{emailService.js, paymentService.js, index.js} src/utils/{logger.js,validator.js,index.js} src/loaders/{dbLoader.js,expressLoader.js,index.js} src/jobs/{emailJob.js,index.js}
touch .env .gitignore README.md Dockerfile
```

### Step 3: Code the Basic Files

1. **`.env`**: Store environment variables here.
   ```plaintext
   PORT=5000
   MONGO_URI=mongodb://localhost:27017/mydatabase
   JWT_SECRET=your-secret-key
   ```

2. **`.gitignore`**: Ignore node_modules and logs.
   ```gitignore
   node_modules/
   logs/
   .env
   ```

3. **`src/app.js`**: Set up your Express app.
   ```js
   const express = require('express');
   const cors = require('cors');
   const morgan = require('morgan');
   const { expressLoader } = require('./loaders/expressLoader');
   
   const app = express();
   
   app.use(cors());
   app.use(express.json());
   app.use(morgan('dev'));  // logging middleware
   
   expressLoader(app);  // Load routes and other things
   
   module.exports = app;
   ```

4. **`src/server.js`**: Main server file.
   ```js
   const app = require('./app');
   const mongoose = require('mongoose');
   const dotenv = require('dotenv');
   const { dbLoader } = require('./loaders/dbLoader');
   
   dotenv.config();
   
   const startServer = async () => {
     try {
       await dbLoader();
       app.listen(process.env.PORT, () => {
         console.log(`Server is running on port ${process.env.PORT}`);
       });
     } catch (err) {
       console.error('Error starting the server:', err);
     }
   };
   
   startServer();
   ```

5. **`src/loaders/expressLoader.js`**: Load Express routes.
   ```js
   const { userRoutes } = require('../routes/userRoutes');
   const { authRoutes } = require('../routes/authRoutes');

   const expressLoader = (app) => {
     app.use('/api/auth', authRoutes);
     app.use('/api/users', userRoutes);
   };

   module.exports = { expressLoader };
   ```

6. **`src/loaders/dbLoader.js`**: Database loader (for MongoDB connection).
   ```js
   const mongoose = require('mongoose');
   const dotenv = require('dotenv');
   
   dotenv.config();
   
   const dbLoader = async () => {
     try {
       await mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true });
       console.log('MongoDB connected');
     } catch (error) {
       console.error('Error connecting to MongoDB:', error);
       process.exit(1);
     }
   };
   
   module.exports = { dbLoader };
   ```

7. **`src/routes/authRoutes.js`**: Auth routes.
   ```js
   const express = require('express');
   const { authController } = require('../controllers/authController');
   
   const router = express.Router();
   
   router.post('/login', authController.login);
   router.post('/register', authController.register);
   
   module.exports = { authRoutes: router };
   ```

8. **`src/routes/userRoutes.js`**: User routes.
   ```js
   const express = require('express');
   const { userController } = require('../controllers/userController');
   
   const router = express.Router();
   
   router.get('/', userController.getAllUsers);
   router.get('/:id', userController.getUserById);
   
   module.exports = { userRoutes: router };
   ```

9. **`src/controllers/authController.js`**: Basic auth controller.
   ```js
   const login = (req, res) => {
     res.send('Login endpoint');
   };

   const register = (req, res) => {
     res.send('Register endpoint');
   };
   
   module.exports = { authController: { login, register } };
   ```

10. **`src/controllers/userController.js`**: Basic user controller.
    ```js
    const getAllUsers = (req, res) => {
      res.send('Fetching all users');
    };

    const getUserById = (req, res) => {
      res.send(`Fetching user with id ${req.params.id}`);
    };
    
    module.exports = { userController: { getAllUsers, getUserById } };
    ```

11. **`src/models/userModel.js`**: Mongoose user model.
    ```js
    const mongoose = require('mongoose');

    const userSchema = new mongoose.Schema({
      name: { type: String, required: true },
      email: { type: String, required: true, unique: true },
      password: { type: String, required: true },
    });

    const User = mongoose.model('User', userSchema);

    module.exports = { User };
    ```

12. **`src/middlewares/errorMiddleware.js`**: Error handling middleware.
    ```js
    const errorMiddleware = (err, req, res, next) => {
      console.error(err.stack);
      res.status(500).json({ message: 'Something went wrong!' });
    };

    module.exports = { errorMiddleware };
    ```

13. **`src/middlewares/authMiddleware.js`**: Authentication middleware (basic JWT example).
    ```js
    const jwt = require('jsonwebtoken');
    
    const authMiddleware = (req, res, next) => {
      const token = req.headers['authorization'];
      if (!token) {
        return res.status(403).json({ message: 'No token provided' });
      }
      
      jwt.verify(token, process.env.JWT_SECRET, (err, decoded) => {
        if (err) {
          return res.status(401).json({ message: 'Unauthorized' });
        }
        req.user = decoded;
        next();
      });
    };
    
    module.exports = { authMiddleware };
    ```

14. **`src/utils/logger.js`**: Logging utility (using Winston).
    ```js
    const winston = require('winston');

    const logger = winston.createLogger({
      level: 'info',
      format: winston.format.simple(),
      transports: [
        new winston.transports.Console(),
        new winston.transports.File({ filename: 'logs/app.log' })
      ]
    });

    module.exports = { logger };
    ```

15. **`Dockerfile`**: For Docker containerization.
    ```dockerfile
    FROM node:16

    WORKDIR /app

    COPY package*.json ./
    RUN npm install

    COPY . .

    EXPOSE 5000
    CMD ["node", "src/server.js"]
    ```

### Step 4: Running the App
1. Create a `.env` file with your environment variables like `MONGO_URI`, `PORT`, and `JWT_SECRET`.
2. Run your app with `npm start` or `nodemon` for development.

This basic setup will provide you with a production-grade structure to start building your backend. You can extend and modify it as needed, adding more advanced features like validation, caching, rate-limiting, and background job processing.
