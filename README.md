# Springboard_Banking_Project
4.5 Python OOP Banking System Mini-Project

Project submission is provided in the form of a jupyter notebook named "bankingproj_springboard.ipynb".
The code works by using the sqlite3 package to establish a local db ("banksys.db") with two tables - "customers" and "accounts":

The "customers" table contains two columns - "custkey" (unique customer identifier) and "name".
The "accounts" table contains four columns - "acctkey" (unique account identifier), "balance", "interest_rate", and "custkey" (used to link accounts to customers).

The "customers" table is populated by information contained within objects of a "Cust" class, which has the attributes "custkey", "name", and "accounts".
The following methods are defined for the "Cust" class: update_db, add_account(), remove_account(), and get_accounts()

The "accounts" table is populated by information contained within objects of the classes "CheckingAccount" and "SavingsAccount" which both have the attributes "balance", "acctkey", "interest_rate", and "custkey"
the following methods are defined for the "CheckingAccount" and "SavingsAccount" classes: update_db(), deposit(), withdraw(), apply_interest(), and check_balance().
Note that accounts can be initialized independently and later linked/unlinked to a customer by using the add_account()/remove_account() methods for "Cust"

to consolidate all of the defined customers within the banking system, a "clientlist" class exists.
The following methods are defined for the "clientlist" class: add_cust(), find_cust(), and transfer() (allows transfering of funds between accounts under the umbrella of the defined clientlist).
