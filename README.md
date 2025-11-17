# Standard-Industry-Database-Design---SQL



/*	Project Objective: To design a standard industry like database that stores informations and other confidential details. As an aspiring Data Engineer.

(1) Give a detailed breakdown on different SQL syntax used during the process, that makes it easy for a non technical audiece to understand.
(2)	Document all process into a pdf or Github for portfolio purposes. 
(3)	Take a screenshot of each table created for portfolio purposes.
(4)	Upload to all social platforms.

*/

# Database concept details

## Data Types
- VARCHAR                    -  Variable length string with (max) characters.
- BOOLEAN                    -  True / False data types.
- CHAR                       -  Specified number of characters with length.
- DECIMAL                    -  Return values in a decimal place
- TEXT                       -  Return unlimited texts in a column.
- INT                        -  Integer values.
- DATE / DATETIME            -  Returns the date or datetime column.
- CURRENT_TIMESTAMP          -  Return the current date of the system running SQL.
- GETDATE                    -  Alternative to the 'CURRENT_TIMESTAMP' function returns the system current date.

# Constraints
- PRIMARY KEY: Unique identifier for each row
- UNIQUE: No duplicate values allowed in this column
- NOT NULL: Column must have a value
- REFERENCES: Foreign key relationship to another table
- ON DELETE CASCADE: Automatically delete related records when parent is deleted

``` sql

----|| Create a Users table to set user profile details || ----



CREATE TABLE UserT (
    UserID              INT PRIMARY KEY IDENTITY(1,1),      --- 'PRIMARY KEY' is a unique identifier of a table (row).
    Username            VARCHAR(50)         NOT NULL,       --- 'VARCHAR' can only contain 50 characters or letters.
    Email               VARCHAR(255)        NOT NULL,
    Phone               VARCHAR(20)         NOT NULL,
    Password            VARCHAR(255)        NOT NULL,
    FirstName           VARCHAR(50)         NOT NULL,
    LastName            VARCHAR(50)         NOT NULL,
    DOB                 DATE                NOT NULL,       --- Returns the 'DATE' data type.
    Gender              VARCHAR(50)         NOT NULL    DEFAULT 'Not Specified'
        CHECK (Gender IN ('Male','Female','Binary','Not Specified')),       --- Can only contain the 'DEFAULT' specified gender.
    ProfilePicture_URL  TEXT,         
    CoverPhoto_URL      TEXT,             
    Bio                 TEXT,
    LanguageCode        VARCHAR(10) DEFAULT 'en',
    Timezone            VARCHAR(50),
    IsVerified          BIT DEFAULT 0           NOT NULL,           --- Can only contain boolean data type of 1 (True) and 0 (False).
    IsActive            BIT DEFAULT 1           NOT NULL,
    LastLogin_At        DATETIME,
    Created_at          DATETIME    DEFAULT     CURRENT_TIMESTAMP,      --- Return the current date and time of the system running SQL.
    Updated_at          DATETIME    DEFAULT     CURRENT_TIMESTAMP,
    Deleted_at          DATETIME
);
``` 

``` sql

----|| Create a 'user authentication providers' table to store log in methods || ----

CREATE TABLE User_Auth_Providers (
    Provider_id         INT     PRIMARY KEY     IDENTITY(1,1),  --- 'IDENTITY' refers to auto increment by 1.
    UserID              INT                     NOT NULL 
        REFERENCES UserT(UserID) ON DELETE CASCADE,
    ProviderType        VARCHAR(50)             NOT NULL,       --- 'NOT NULL' - column cannot be empty.
    Provider_userID     VARCHAR(250)            NOT NULL,
    Access_Token        TEXT,
    Refresh_Token       TEXT,
    Token_expiresAt     DATETIME,
    UNIQUE (ProviderType, Provider_userID)          --- Prevents duplicates connections.
);
```

```sql 

----|| Create 'login attempts' table for security monitoring to enable fraud detection || ----

CREATE TABLE Login_Attempts (
    Attempt_id          INT     PRIMARY KEY     IDENTITY(1,1),
    UserID              INT                     NOT NULL 
        REFERENCES UserT(UserID) ON DELETE CASCADE,                 --- 'ON DELETE CASACDE' refers to if infomation is deleted from the 'UserT' table (parental table) any related info will also be deleted..
    Ip_address          VARCHAR(45)             NOT NULL,
    User_agent          TEXT,
    Attempt_status      VARCHAR(20)             NOT NULL,
    Failure_reason      VARCHAR(100),
    Attempted_at        DATETIME    DEFAULT     CURRENT_TIMESTAMP
);
```


``` sql

----|| Create 'Payment Method' table for financial instruments  || ----

CREATE TABLE Payment_Method (
    Payment_Method_id   INT     PRIMARY KEY     IDENTITY(1,1),
    UserID              INT                     NOT NULL 
        REFERENCES UserT(UserID) ON DELETE CASCADE,
    MethodType          VARCHAR(50)             NOT NULL,
    Provider_Name       VARCHAR(100)            NOT NULL,
    AccountIdentifier   VARCHAR(255),
    Is_Primary          BIT DEFAULT 0,
    Is_Verified         BIT DEFAULT 0,
    VerificationDate    DATE,
    Metadata            VARCHAR(45),
    Created_at          DATETIME    DEFAULT     CURRENT_TIMESTAMP,
    Updated_at          DATETIME    DEFAULT     CURRENT_TIMESTAMP,
    Expires_at          DATETIME                    --- Returns the datetime data type.
);

```

```sql

----|| Create 'CreditCard' table for card specific data  || ----

CREATE TABLE CreditCardDetails (
    Card_id             INT     PRIMARY KEY     IDENTITY(1,1),
    Payment_Method_id   INT                     NOT NULL 
        REFERENCES Payment_Method(Payment_Method_id) ON DELETE CASCADE,
    CardHolderName      TEXT,
    CardType            VARCHAR(255) DEFAULT 'Verve'
        CHECK (CardType IN ('Verve','Mastercard','Bank Transfer','USDT','Cash Deposit')),
    LastFourDigits      CHAR(4),
    Expiry_Month        INT                     NOT NULL,           --- Returns 'INTEGER' data type.
    ExpiryYear          INT                     NOT NULL,
    BillingAddress_id   INT,    
    Created_at          DATETIME    DEFAULT     CURRENT_TIMESTAMP
);

```


``` sql

----|| Create 'AddressDetails' table to store for location management  || ----

CREATE TABLE AddressDetails (
    Address_id          INT     PRIMARY KEY     IDENTITY(1,1),
    UserID              INT             NOT NULL 
        REFERENCES UserT(UserID)    ON DELETE CASCADE,
    Address_type        VARCHAR(50),
    Street_Address1     VARCHAR(250)    NOT NULL,
    Street_Address2     VARCHAR(250),
    City                VARCHAR(100)    NOT NULL,
    State_province      VARCHAR(50),
    Postal_code         VARCHAR(20)     NOT NULL,
    CountryCode         CHAR(2)         NOT NULL,
    Is_Primary          BIT DEFAULT 0   NOT NULL,
    Latitude            DECIMAL(10,7),                  --- 'DECIMAL' values can only be a total of 10 (7 digits after decimal).
    Longitude           DECIMAL(11,8),
    Created_at          DATETIME    DEFAULT     (GETDATE()),
    Updated_at          DATETIME    DEFAULT     (GETDATE())     --- Returns the current date on the system. An alternative to 'CURRENT_TIMESTAMP'.
);

```

``` sql

----|| Create 'UserPreferences' table to retain information on settings and customization  || ----

CREATE TABLE UserPreferences (
    PreferencesID       INT     PRIMARY KEY     IDENTITY(1,1),
    UserID              INT NOT NULL 
        REFERENCES UserT(UserID)    ON DELETE CASCADE,
    Preferences_type    VARCHAR(50),
    Preferences_value   TEXT,                           --- Returns unlimited number of texts.
    Created_at          DATETIME    DEFAULT     CURRENT_TIMESTAMP,
    Updated_at          DATETIME    DEFAULT     CURRENT_TIMESTAMP,
    UNIQUE (UserID, Preferences_type)
);

```

``` sql
----|| Create 'UserSessions' table to monitor and track user active login sessions || ----

CREATE TABLE UserSessions (
    Session_id          INT PRIMARY KEY IDENTITY(1,1),
    UserID              INT NOT NULL 
        REFERENCES UserT(UserID)    ON DELETE CASCADE,
    Device_id           VARCHAR(255),
    Device_type         VARCHAR(50),
    Devices_OS          VARCHAR(50),
    Browser_name        TEXT,
    IP_Address          VARCHAR(45),
    User_Agent          VARCHAR(255),
    Login_at            DATETIME    DEFAULT     CURRENT_TIMESTAMP,
    Last_activity_at    DATETIME    DEFAULT     CURRENT_TIMESTAMP,
    Expires_at          DATETIME,
    Logout_at           DATETIME,
    Is_Active           BIT DEFAULT 1               --- Returns columns as boolean 1(True).
);
```

```sql

----|| Create 'AuditLogs' table to security and compliance || ----

CREATE TABLE AuditLogs (
    Audit_id            INT     PRIMARY KEY     IDENTITY(1,1),      --- Auto increment by 1.
    UserID              INT         NOT NULL 
        REFERENCES UserT(UserID)    ON DELETE CASCADE,
    Action_type         VARCHAR(50),
    Table_name          VARCHAR(100),
    Record_cd           TEXT,
    Old_Values          VARCHAR(50),
    New_Values          VARCHAR(50),
    IP_Address          VARCHAR(45),
    User_agent          TEXT,
    Performed_at        DATETIME DEFAULT CURRENT_TIMESTAMP
);

```
