```python
import sqlite3

# Connect to the database (or create it if it doesn't exist)
con = sqlite3.connect('banksys.db')

# Create a cursor object to interact with the database
c = con.cursor()
```


```python
def initialize():
    c.execute('''
    CREATE TABLE IF NOT EXISTS customers (
        custkey INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL
    )
    ''')

    c.execute('''
    CREATE TABLE IF NOT EXISTS  accounts (
        acctkey INTEGER PRIMARY KEY,
        balance REAL NOT NULL,
        interest_rate REAL DEFAULT 0.0,
        custkey INTEGER,
        FOREIGN KEY (custkey) REFERENCES customers (custkey)
    )
    ''')

    # Commit changes
    con.commit()

initialize()
```


```python
class Cust:
    def __init__(self, name, custkey=None):
        self.custkey = custkey
        self.name = name
        self.accounts = []

    def update_db(self):
        if self.custkey is None:
            c.execute('INSERT INTO customers (name) VALUES (?)', (self.name,))
            self.custkey = c.lastrowid
        else:
            c.execute('UPDATE customers SET name = ? WHERE custkey = ?', (self.name, self.custkey))
        con.commit()

    def add_account(self, account):
        self.accounts.append(account)
        account.custkey = self.custkey
        account.update_db()
        print(f"Account {account.acctkey} added to customer {self.name}")

    def remove_account(self, acctkey):
        self.accounts = [acc for acc in self.accounts if acc.acctkey != acctkey]
        c.execute('DELETE FROM accounts WHERE acctkey = ?', (acctkey,))
        con.commit()
        print(f"Account {acctey} removed from customer {self.name}")

    def get_accounts(self):
        c.execute('SELECT acctkey, balance, interest_rate FROM accounts WHERE custkey = ?', (self.custkey,))
        rows = c.fetchall()
        self.accounts = [CheckingAccount(balance=row[1], acctkey=row[0], interest_rate=row[2]) for row in rows]
        return self.accounts
```


```python
class CheckingAccount:
    acctkey_counter = 8000

    def __init__(self, balance=0, acctkey=None, interest_rate=0.0, custkey=None):
        self.acctkey = acctkey if acctkey is not None else CheckingAccount.acctkey_counter
        CheckingAccount.acctkey_counter += 1
        self.balance = balance
        self.interest_rate = interest_rate
        self.custkey = custkey

    def update_db(self):
        c.execute('''
        INSERT OR REPLACE INTO accounts (acctkey, balance, interest_rate, custkey)
        VALUES (?, ?, ?, ?)
        ''', (self.acctkey, self.balance, self.interest_rate, self.custkey))
        con.commit()

    def deposit(self, amount):
        if amount > 0:
            self.balance += amount
            self.update_db()
            print(f"Deposited {amount}. New balance: {self.balance}")
        else:
            print("Deposit amount must be positive")

    def withdraw(self, amount):
        if 0 < amount <= self.balance:
            self.balance -= amount
            self.update_db()
            print(f"Withdrew {amount}. New balance: {self.balance}")
        else:
            print("Insufficient funds or invalid amount")
            
    def apply_interest(self):
        interest = self.balance * self.interest_rate
        self.balance += interest
        self.update_db()
        print(f"Applied interest {interest}. New balance: {self.balance}")
    
    def check_balance(self):
        return self.balance

class SavingsAccount(CheckingAccount):
    def __init__(self, balance=0, interest_rate=0.01, acctkey=None, custkey=None):
        super().__init__(balance, acctkey, interest_rate, custkey)
```


```python
class clientlist:
    def __init__(self):
        self.customers = []

    def add_cust(self, cust):
        cust.update_db()
        self.customers.append(cust)
        print(f"Customer {cust.name} added to client list")

    def find_cust(self, custkey):
        c.execute('SELECT custkey, name FROM customers WHERE custkey = ?', (custkey,))
        row = c.fetchone()
        if row:
            cust = Cust(row[1], row[0])
            cust.get_accounts()
            return cust
        return None

    def transfer(self, from_account, to_account, amount):
        if from_account.balance >= amount:
            from_account.withdraw(amount)
            to_account.deposit(amount)
            print(f"Transferred {amount} from account {from_account.acctkey} to account {to_account.acctkey}")
        else:
            print("Insufficient funds for transfer")
```


```python
# Create bank instance
clients = clientlist()

# Create sample customers
custa = Cust("Murat")
custb = Cust("Eric")
custc = Cust("Fred")

# Add customers to clientlist
clients.add_cust(custa)
clients.add_cust(custb)
clients.add_cust(custc)

# Create accounts for customers
account1 = CheckingAccount()
account2 = SavingsAccount(interest_rate=0.05)
account3 = CheckingAccount(interest_rate=0.01)
account4 = SavingsAccount(interest_rate=0.06)


# Add accounts to users
custa.add_account(account1)
custb.add_account(account2)
custc.add_account(account3)
custc.add_account(account4)

# Perform transactions
account1.deposit(1000)
account1.apply_interest()
account1.withdraw(500)
account2.deposit(2000)
account3.deposit(5000)
account3.apply_interest()
account4.deposit(10000)
account4.apply_interest()

# Transfer money
clients.transfer(account1, account2, 200)

# Retrieve user and account information
retrieved_custa = clients.find_cust(custa.custkey)
retrieved_custb = clients.find_cust(custb.custkey)
retrieved_custc = clients.find_cust(custc.custkey)

print(f"User: {retrieved_custa.name}, Accounts: {[acc.acctkey for acc in retrieved_custa.get_accounts()]}")
print(f"User: {retrieved_custb.name}, Accounts: {[acc.acctkey for acc in retrieved_custb.get_accounts()]}")
print(f"User: {retrieved_custc.name}, Accounts: {[acc.acctkey for acc in retrieved_custc.get_accounts()]}")


```

    Customer Murat added to client list
    Customer Eric added to client list
    Customer Fred added to client list
    Account 8000 added to customer Murat
    Account 8001 added to customer Eric
    Account 8002 added to customer Fred
    Account 8003 added to customer Fred
    Deposited 1000. New balance: 1000
    Applied interest 0.0. New balance: 1000.0
    Withdrew 500. New balance: 500.0
    Deposited 2000. New balance: 2000
    Deposited 5000. New balance: 5000
    Applied interest 50.0. New balance: 5050.0
    Deposited 10000. New balance: 10000
    Applied interest 600.0. New balance: 10600.0
    Withdrew 200. New balance: 300.0
    Deposited 200. New balance: 2200
    Transferred 200 from account 8000 to account 8001
    User: Murat, Accounts: [8000]
    User: Eric, Accounts: [8001]
    User: Fred, Accounts: [8002, 8003]
    


```python
import pandas as pd
customers_df = pd.read_sql_query("SELECT * FROM customers", con)
print(customers_df)
accounts_df = pd.read_sql_query("SELECT * FROM accounts", con)
print(accounts_df)
```

       custkey   name
    0        1  Murat
    1        2   Eric
    2        3   Fred
       acctkey  balance  interest_rate  custkey
    0     8000    300.0           0.00        1
    1     8001   2200.0           0.05        2
    2     8002   5050.0           0.01        3
    3     8003  10600.0           0.06        3
    


```python

```
