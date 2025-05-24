# Simple Banking App v2 Security Enhanced

## Group Members

- Garcia, Asi Neo
- Mediado, Hyacinthe
- Janda, Christian Paul

## Introduction

This project is a security-enhanced version of the Simple Banking App, a Flask-based web application simulating basic banking functionalities. This iteration focuses on identifying and addressing potential security vulnerabilities to improve the application's robustness, integrity, and user data protection.

## Objectives

- To enhance the security posture of the original Simple Banking App.
- To implement measures to protect against common web vulnerabilities.
- To improve data integrity for sensitive operations.
- To provide a clearer audit trail for administrative actions.
- To document the identified vulnerabilities and the implemented solutions.

## Original Application Features

- **User Authentication**
  - Secure login with username/password
  - Registration of new users
  - Password recovery mechanism (email-based)

- **Account Management**
  - Display of account balance
  - View recent transaction history (last 10 transactions)

- **Fund Transfer**
  - Transfer money to other registered users
  - Confirmation screen before completing transfers
  - Transaction history updated after transfers

- **User Role Management**
  - Regular user accounts
  - Admin users with account approval capabilities
  - Manager users who can manage admin accounts

- **Location Data Integration**
  - Philippine Standard Geographic Code (PSGC) API integration
  - Hierarchical location data selection (Region, Province, City, Barangay)
  - Form fields pre-populated with location data

- **Admin Features**
  - User account approval workflow
  - Account activation/deactivation
  - Deposit funds to user accounts
  - Create new accounts
  - Edit user information

- **Manager Features**
  - Create and manage admin accounts
  - View admin transaction logs
  - Monitor all system transfers

## Security Assessment Findings

Based on the review of the original application code, the following areas were identified as potential vulnerabilities or areas for security improvement:

-   **Data Integrity in Financial Transactions:** Lack of explicit transaction handling in critical operations like money transfers and deposits could lead to inconsistent data in case of errors.
-   **Input Validation:** While basic validation was present, more comprehensive checks on sensitive inputs (e.g., amounts, account numbers, password complexity) could prevent potential issues.
-   **Logging of Administrative Actions:** Insufficient logging of administrative modifications to user accounts could hinder auditing and accountability.
-   **Visibility of Sensitive Information:** Displaying sensitive information like full account balance and number by default on the account page could be a privacy concern.
-   **(Potential) Missing CSRF Protection on Dynamically Triggered Actions:** Actions initiated via JavaScript without proper CSRF token handling could be vulnerable.

## Security Improvements Implemented

This section outlines the specific updates made to the application to address the identified areas and enhance its security posture:

-   **Improved Financial Transaction Integrity:** Added explicit database transaction handling to money transfer and deposit operations (`models.py`) using `try...except...db.session.rollback()` blocks. This ensures that these critical operations are atomic, guaranteeing data consistency.
-   **Stricter Form Input Validation:** Implemented more robust validation in `forms.py`. This includes:
    -   Adding `Length` and `Regexp` validators to the `RegistrationForm` password field to enforce a minimum length of 8 characters and require a combination of uppercase letters, lowercase letters, digits, and symbols.
    -   Adding `Length` and `Regexp` validators to account number fields in `TransferForm` and `DepositForm` to ensure they are 10-digit numeric strings.
    -   Added placeholder validation for maximum transfer and deposit amounts in `TransferForm` and `DepositForm`.
-   **Administrative Action Logging:** Introduced comprehensive logging for sensitive administrative actions in `routes.py`. User activation (`/admin/activate_user`), deactivation (`/admin/deactivate_user`), and profile edits (`/admin/edit_user`) now create detailed `Transaction` records with a specific type and details about the changes made, providing an essential audit trail.
-   **Masked Sensitive Information:** Enhanced the account page (`templates/account.html`) to mask the user's current balance and account number by default using asterisks. JavaScript toggle buttons were added next to these elements, allowing users to reveal the sensitive information on demand, improving privacy.
-   **Addressed CSRF on Dynamic Actions:** Ensured that actions triggered by JavaScript (like the delete history function, which was later removed but the fix implemented) include a CSRF token by embedding the button within a form that includes `{{ csrf_token() }}`.

## Penetration Testing Report Summary

The following steps were taken to remediate the identified potential vulnerabilities:

-   Implemented atomic transactions for financial operations.
-   Added stricter server-side input validation for critical forms.
-   Introduced detailed logging for administrative actions.
-   Modified frontend templates to mask sensitive user information by default.
-   Ensured CSRF tokens are included in dynamically triggered POST requests.

## Technology Stack

- **Backend**: Python, Flask
- **Database**: MySQL (with SQLAlchemy ORM)
- **Frontend**: HTML, CSS, Bootstrap 5, JavaScript
- **Authentication**: Flask-Login, Werkzeug, Flask-Bcrypt, `itsdangerous` (for token generation)
- **Forms**: Flask-WTF, WTForms
- **Security**: Flask-Limiter (for API rate limiting), Flask-WTF (for CSRF protection and optional Recaptcha)
- **External API**: PSGC API for Philippine geographic data

## Setup Instructions

### Prerequisites

- Python 3.7+
- pip (Python package manager)
- MySQL Server 5.7+ or MariaDB 10.2+

### Database Setup

1.  Install MySQL Server or MariaDB if you haven't already. Refer to the official documentation for your operating system.

2.  Create a database and a user with privileges for this database. Replace `'bankapp'` and `'your_password'` with your desired username and password:
    ```sql
    CREATE DATABASE simple_banking_v2;
    CREATE USER 'bankapp'@'localhost' IDENTIFIED BY 'your_password';
    GRANT ALL PRIVILEGES ON simple_banking_v2.* TO 'bankapp'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;
    ```

3.  Create a `.env` file in the root directory of the project and add your database credentials. You also need to set a `SECRET_KEY` for Flask.
    ```dotenv
    SECRET_KEY=your_secret_key_here # Replace with a strong, random key
    MYSQL_USER=bankapp
    MYSQL_PASSWORD=your_password
    MYSQL_HOST=localhost
    MYSQL_PORT=3306
    MYSQL_DATABASE=simple_banking_v2
    # Optional: for Redis rate limiting persistence
    # REDIS_URL=redis://localhost:6379/0
    # Optional: for Google reCAPTCHA (if implemented)
    # RECAPTCHA_PUBLIC_KEY=your_site_key
    # RECAPTCHA_PRIVATE_KEY=your_secret_key
    ```
    Make sure to replace `'your_secret_key_here'`, `'bankapp'`, `'your_password'`, and update database details as needed.

4.  Initialize the database tables and create a default admin user by running:
    ```bash
    python init_db.py
    ```

### Installation

1.  Clone the repository or download the code.

2.  Navigate to the project directory in your terminal.
    ```bash
    cd simple-banking-app-v2
    ```

3.  Install the required Python packages using pip:
    ```bash
    pip install -r requirements.txt
    ```

### Running the Application

1.  Ensure your MySQL server is running.

2.  Run the Flask application:
    ```bash
    python app.py
    ```

3.  Access the application through your web browser at `http://localhost:5000`.

## Usage

- Follow the sections in "Original Application Features" and note the security enhancements implemented.
- The login and registration processes now have stricter validation.
- The account page masks sensitive details by default.
- Administrative actions are logged for auditing.

## User Roles

The system supports three types of user roles:

1. **Regular Users** - Can manage their own account, make transfers, and view their transaction history.

2. **Admin Users** - Have all regular user privileges plus:
   - Approve/reject new user registrations
   - Activate/deactivate user accounts
   - Create new user accounts
   - Make deposits to user accounts
   - Edit user information

3. **Manager Users** - Have all admin privileges plus:
   - Create and manage admin accounts
   - View admin transaction logs
   - Monitor all system transfers
   - System-wide oversight capabilities

## Address Management with PSGC API

The application integrates with the Philippine Standard Geographic Code (PSGC) API to provide standardized address selection for user profiles. The address system follows the Philippine geographical hierarchy:

- Region
- Province
- City/Municipality
- Barangay

This integration ensures addresses are standardized and validates location data according to the Philippine geographical structure.

## Rate Limiting

The application uses Flask-Limiter to implement API rate limiting, which protects against potential DoS attacks and abusive bot activity. The rate limits are configured as follows:

- **Login**: 10 attempts per minute
- **Registration**: 5 attempts per minute
- **Password Reset**: 5 attempts per hour
- **Money Transfer**: 20 attempts per hour
- **API Endpoints**: 30 requests per minute
- **Admin Dashboard**: 60 requests per hour
- **Admin Account Creation**: 20 accounts per hour
- **Admin Deposits**: 30 deposits per hour
- **Manager Dashboard**: 60 requests per hour
- **Admin Creation**: 10 admin accounts per hour

By default, the rate limiting data is stored in memory. For production use, it's recommended to use Redis as a storage backend for persistence across application restarts. To enable Redis storage:

1. Install Redis server on your system
2. Update the `.env` file with your Redis URL:
   ```
   REDIS_URL=redis://localhost:6379/0
   ```

If Redis is not available, the application will automatically fall back to in-memory storage.

## Deploying to PythonAnywhere

1. Create a PythonAnywhere account at [www.pythonanywhere.com](https://www.pythonanywhere.com)

2. Upload your code using Git:
   ```
   git clone https://github.com/yourusername/simple-banking-app-v2.git
   ```

3. Install requirements:
   ```
   cd simple-banking-app-v2
   pip install -r requirements.txt
   ```

4. Set up MySQL database on PythonAnywhere:
   - Go to the Databases tab in your PythonAnywhere dashboard
   - Create a new MySQL database
   - Note the database name, username, and password
   - Update your .env file with these credentials

5. Initialize your database on PythonAnywhere:
   ```
   python init_db.py
   ```

6. Configure a new web app via the PythonAnywhere dashboard:
   - Select "Manual configuration"
   - Choose Python 3.8
   - Set source code directory to `/home/yourusername/simple-banking-app-v2`
   - Set working directory to `/home/yourusername/simple-banking-app-v2`
   - Set WSGI configuration file to point to your Flask app

7. Add environment variables in the PythonAnywhere dashboard for security

## License

This project is licensed under the MIT License - see the LICENSE file for details.
