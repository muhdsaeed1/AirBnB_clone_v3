
## DEVELOPING RESTFUL API USING PYTHON AND FLASK

Nowadays, choosing Python to develop applications is becoming a very popular choice.

When it comes to web development on Python, there are three predominant frameworks: Django, Flask, and a relatively new player FastAPI. Django is older, more mature, and a little bit more popular

Bootstrapping a Flask Application
First and foremost, we will need to install some dependencies on our development machine. We will need to install Python 3, Pip (Python Package Index), and Flask.

Installing Python 3
If we are using some recent version of a popular Linux distribution (like Ubuntu) or macOS, we might already have Python 3 installed on our computer. If we are running Windows, we will probably need to install Python 3, as this operating system does not ship with any version.

After installing Python 3 on our machine, we can check that we have everything set up as expected by running the following command:

python --version
# Python 3.8.9
Note that the command above might produce a different output when we have a different Python version. What is important is that you are running at least Python 3.7 or newer. If we get "Python 2" instead, we can try issuing python3 --version. If this command produces the correct output, we must replace all commands throughout the article to use python3 instead of just python.

Installing Pip
Pip is the recommended tool for installing Python packages. While the official installation page states that pip comes installed if we're using Python 2 >= 2.7.9 or Python 3 >= 3.4, installing Python through apt on Ubuntu doesn't install pip. Therefore, let's check if we need to install pip separately or already have it.

# we might need to change pip by pip3
pip --version
# pip 9.0.1 ... (python 3.X)
If the command above produces an output similar to pip 9.0.1 ... (python 3.X), then we are good to go. If we get pip 9.0.1 ... (python 2.X), we can try replacing pip with pip3. If we cannot find Pip for Python 3 on our machine, we can follow the instructions here to install Pip.

Installing Flask
We already know what Flask is and its capabilities. Therefore, let's focus on installing it on our machine and testing to see if we can get a basic Flask application running. The first step is to use pip to install Flask:

# we might need to replace pip with pip3
pip install Flask
After installing the package, we will create a file called hello.py and add five lines of code to it. As we will use this file to check if Flask was correctly installed, we don't need to nest it in a new directory.

# hello.py

from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "Hello, World!"
These 5 lines of code are everything we need to handle HTTP requests and return a "Hello, World!" message. To run it, we execute the following command:

flask --app hello run

 * Serving Flask app 'hello'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit

Virtual environments (virtualenv)
Although PyPA—the Python Packaging Authority group—recommends pip as the tool for installing Python packages, we will need to use another package to manage our project's dependencies. It's true that pip supports package management through the requirements.txt file, but the tool lacks some features required on serious projects running on different production and development machines. Among its issues, the ones that cause the most problems are:

pip installs packages globally, making it hard to manage multiple versions of the same package on the same machine.
requirements.txt need all dependencies and sub-dependencies listed explicitly, a manual process that is tedious and error-prone.
To solve these issues, we are going to use Pipenv. Pipenv is a dependency manager that isolates projects in private environments, allowing packages to be installed per project. If you're familiar with NPM or Ruby's bundler, it's similar in spirit to those tools.

pip install pipenv
Now, to start creating a serious Flask application, let's create a new directory that will hold our source code. In this article, we will create Cashman, a small RESTful API that allows users to manage incomes and expenses. Therefore, we will create a directory called cashman-flask-project. After that, we will use pipenv to start our project and manage our dependencies.

# create our project directory and move to it
mkdir cashman-flask-project && cd cashman-flask-project

# use pipenv to create a Python 3 (--three) virtualenv for our project
pipenv --three

# install flask a dependency on our project
pipenv install flask
The second command creates our virtual environment, where all our dependencies get installed, and the third will add Flask as our first dependency. If we check our project's directory, we will see two new files:

Pipfile contains details about our project, such as the Python version and the packages needed.
Pipenv.lock contains precisely what version of each package our project depends on and its transitive dependencies.
Python modules
Like other mainstream programming languages, Python also has the concept of modules to enable developers to organize source code according to subjects/functionalities. Similar to Java packages and C# namespaces, modules in Python are files organized in directories that other Python scripts can import. To create a module on a Python application, we need to create a folder and add an empty file called __init__.py.

Let's create our first module on our application, the main module, with all our RESTful endpoints. Inside the application's directory, let's create another one with the same name, cashman. The root cashman-flask-project directory created before will hold metadata about our project, like what dependencies it has, while this new one will be our module with our Python scripts.

# create source code's root
mkdir cashman && cd cashman

# create an empty __init__.py file
touch __init__.py
Inside the main module, let's create a script called index.py. In this script, we will define the first endpoint of our application.

from flask import Flask
app = Flask(__name__)


@app.route("/")
def hello_world():
    return "Hello, World!"
As in the previous example, our application returns a "Hello, world!" message. We will start improving it in a second, but first, let's create an executable file called bootstrap.sh in the root directory of our application.

# move to the root directory
cd ..

# create the file
touch bootstrap.sh

# make it executable
chmod +x bootstrap.sh
The goal of this file is to facilitate the start-up of our application. Its source code will be the following:

#!/bin/sh
export FLASK_APP=./cashman/index.py
pipenv run flask --debug run -h 0.0.0.0
The first command defines the main script to be executed by Flask. The second command runs our Flask application in the context of the virtual environment listening to all interfaces on the computer (-h 0.0.0.0).

Note: we are setting flask to run in debug mode to enhance our development experience and activate the hot reload feature, so we don't have to restart the server each time we change the code. If you run Flask in production, we recommend updating these settings for production.

To check that this script is working correctly, we run ./bootstrap.sh to get similar results as when executing the "Hello, world!" application.

 * Serving Flask app './cashman/index.py'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://192.168.1.207:5000
Press CTRL+C to quit
Creating a RESTful Endpoint with Flask
Now that our application is structured, we can start coding some relevant endpoints. As mentioned before, the goal of our application is to help users to manage incomes and expenses. We will begin by defining two endpoints to handle incomes. Let's replace the contents of the ./cashman/index.py file with the following:

from flask import Flask, jsonify, request

app = Flask(__name__)

incomes = [
    { 'description': 'salary', 'amount': 5000 }
]


@app.route('/incomes')
def get_incomes():
    return jsonify(incomes)


@app.route('/incomes', methods=['POST'])
def add_income():
    incomes.append(request.get_json())
    return '', 204

    To interact with both endpoints that we have created, we can start our application and issue some HTTP requests:

# start the cashman application
./bootstrap.sh &

# get incomes
curl http://localhost:5000/incomes

# add new income
curl -X POST -H "Content-Type: application/json" -d '{
  "description": "lottery",
  "amount": 1000.0
}' http://localhost:5000/incomes

# check if lottery was added
curl localhost:5000/incomes

Mapping Models with Python Classes
Using dictionaries in a simple use case like the one above is enough. However, for more complex applications that deal with different entities and have multiple business rules and validations, we might need to encapsulate our data into Python classes.

We will refactor our application to learn the process of mapping entities (like incomes) as classes. The first thing that we will do is create a submodule to hold all our entities. Let's create a model directory inside the cashman module and add an empty file called __init__.py on it.

# create model directory inside the cashman module
mkdir -p cashman/model

# initialize it as a module
touch cashman/model/__init__.py
Mapping a Python superclass
We will create three classes in this new module/directory: Transaction, Income, and Expense. The first class will be the base for the two others, and we will call it Transaction. Let's create a file called transaction.py in the model directory with the following code:

import datetime as dt

from marshmallow import Schema, fields


class Transaction(object):
    def __init__(self, description, amount, type):
        self.description = description
        self.amount = amount
        self.created_at = dt.datetime.now()
        self.type = type

    def __repr__(self):
        return '<Transaction(name={self.description!r})>'.format(self=self)


class TransactionSchema(Schema):
    description = fields.Str()
    amount = fields.Number()
    created_at = fields.Date()
    type = fields.Str()
Besides the Transaction class, we also defined a TransactionSchema. We will use the latter to deserialize and serialize instances of Transaction from and to JSON objects. This class inherits from another superclass called Schema that belongs on a package not yet installed.

# installing marshmallow as a project dependency
pipenv install marshmallow
Marshmallow is a popular Python package for converting complex datatypes, such as objects, to and from built-in Python datatypes. We can use this package to validate, serialize, and deserialize data. We won't dive into validation in this article, as it will be the subject of another one. Though, as mentioned, we will use marshmallow to serialize and deserialize entities through our endpoints.

Mapping Income and Expense as Python Classes
To keep things more organized and meaningful, we won't expose the Transaction class on our endpoints. We will create two specializations to handle the requests: Income and Expense. Let's make a file called income.py inside the model module with the following code:

from marshmallow import post_load

from .transaction import Transaction, TransactionSchema
from .transaction_type import TransactionType


class Income(Transaction):
    def __init__(self, description, amount):
        super(Income, self).__init__(description, amount, TransactionType.INCOME)

    def __repr__(self):
        return '<Income(name={self.description!r})>'.format(self=self)


class IncomeSchema(TransactionSchema):
    @post_load
    def make_income(self, data, **kwargs):
        return Income(**data)
The only value that this class adds for our application is that it hardcodes the type of transaction. This type is a Python enumerator, which we still have to create, that will help us filter transactions in the future. Let's create another file, called transaction_type.py, inside model to represent this enumerator:

from enum import Enum


class TransactionType(Enum):
    INCOME = "INCOME"
    EXPENSE = "EXPENSE"
The code of the enumerator is quite simple. It just defines a class called TransactionType that inherits from Enum and that defines two types: INCOME and EXPENSE.

Lastly, let's create the class that represents expenses. To do that, let's add a new file called expense.py inside model with the following code:

from marshmallow import post_load

from .transaction import Transaction, TransactionSchema
from .transaction_type import TransactionType


class Expense(Transaction):
    def __init__(self, description, amount):
        super(Expense, self).__init__(description, -abs(amount), TransactionType.EXPENSE)

    def __repr__(self):
        return '<Expense(name={self.description!r})>'.format(self=self)


class ExpenseSchema(TransactionSchema):
    @post_load
    def make_expense(self, data, **kwargs):
        return Expense(**data)
Similar to Income, this class hardcodes the type of the transaction, but now it passes EXPENSE to the superclass. The difference is that it transforms the given amount to be negative. Therefore, no matter if the user sends a positive or a negative value, we will always store it as negative to facilitate calculations.

Serializing and Deserializing Objects with Marshmallow
With the Transaction superclass and its specializations adequately implemented, we can now enhance our endpoints to deal with these classes. Let's replace ./cashman/index.py contents to:

from flask import Flask, jsonify, request

from cashman.model.expense import Expense, ExpenseSchema
from cashman.model.income import Income, IncomeSchema
from cashman.model.transaction_type import TransactionType

app = Flask(__name__)

transactions = [
    Income('Salary', 5000),
    Income('Dividends', 200),
    Expense('pizza', 50),
    Expense('Rock Concert', 100)
]


@app.route('/incomes')
def get_incomes():
    schema = IncomeSchema(many=True)
    incomes = schema.dump(
        filter(lambda t: t.type == TransactionType.INCOME, transactions)
    )
    return jsonify(incomes)


@app.route('/incomes', methods=['POST'])
def add_income():
    income = IncomeSchema().load(request.get_json())
    transactions.append(income)
    return "", 204


@app.route('/expenses')
def get_expenses():
    schema = ExpenseSchema(many=True)
    expenses = schema.dump(
        filter(lambda t: t.type == TransactionType.EXPENSE, transactions)
    )
    return jsonify(expenses)


@app.route('/expenses', methods=['POST'])
def add_expense():
    expense = ExpenseSchema().load(request.get_json())
    transactions.append(expense)
    return "", 204


if __name__ == "__main__":
    app.run()
