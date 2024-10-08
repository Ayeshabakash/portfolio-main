### Step-by-Step Documentation for the Streamlit Text-to-SQL App 

This guide explains how the Streamlit application is set up, step-by-step. The app converts a natural language question into an SQL query using Google's Gemini AI model and retrieves relevant data from an SQLite database.

#### **Step 1: Import Required Libraries**
The application imports the necessary libraries to make the app function:
```python
from dotenv import load_dotenv
import streamlit as st
import os
import sqlite3
import google.generativeai as genai
```
- **`dotenv`**: Used to load environment variables from a `.env` file.
- **`streamlit`**: This is the framework for building web apps.
- **`os`**: Helps load the environment variables for configuring API keys.
- **`sqlite3`**: Allows interaction with the SQLite database.
- **`google.generativeai`**: Interacts with Google's Gemini AI for natural language processing.

#### **Step 2: Load Environment Variables**
The `load_dotenv()` function is used to load the API key stored in a `.env` file for connecting to Gemini AI.
```python
load_dotenv()
```

#### **Step 3: Configure Gemini API**
This step initializes the Gemini AI by configuring it with the API key:
```python
genai.configure(api_key=os.getenv("GOOGLE_API_KEY"))
```

#### **Step 4: Define Function to Get Gemini Response**
This function takes in a user's question and a predefined prompt to generate an SQL query using Gemini AI. The AI processes the prompt and question and returns the SQL query.

```python
def get_gemini_response(question, prompt):
    model = genai.GenerativeModel('gemini-pro')
    response = model.generate_content([prompt[0], question])
    return response.text
```

#### **Step 5: Define Function to Execute SQL Query**
This function executes the generated SQL query on the local SQLite database and retrieves the resulting data.
```python
def read_sql_query(sql, db):
    conn = sqlite3.connect(db)
    cur = conn.cursor()
    cur.execute(sql)
    rows = cur.fetchall()
    conn.commit()
    conn.close()
    return rows
```
- **`sql`**: The SQL query generated by Gemini AI.
- **`db`**: The SQLite database file (e.g., `student.db`).

#### **Step 6: Define the Predefined Prompt**
This prompt provides the AI with context on how to convert English questions to SQL queries. The database has predefined columns such as `NAME`, `CLASS`, `SECTION`, and `MARK`.

```python
prompt = [
    """
  You are an expert in converting English questions to SQL query!
    The SQL database has the name STUDENT and has the following columns - NAME, CLASS, 
    SECTION and MARK \n\nFor example,\nExample 1 - How many entries of records are present?, 
    the SQL command will be something like this SELECT COUNT(*) FROM STUDENT ;
    \nExample 2 - Tell me all the students studying in Data Science class?, 
    the SQL command will be something like this SELECT * FROM STUDENT 
    where CLASS="Data Science"; 
    also the sql code should not have ``` in beginning or end and sql word in output
    """
]
```

#### **Step 7: Build the Streamlit Interface**
The Streamlit interface allows users to input their natural language query, which is passed to Gemini AI and then executed against the SQLite database. The retrieved data is then displayed on the web app.

- Set the page title and header:
  ```python
  st.set_page_config(page_title="Text to SQL LLM")
  st.header("Gemini AI Powered Application to retrieve SQL Data by natural Language")
  ```

- Create a text input field for users to enter their question:
  ```python
  question = st.text_input("Input : ", key="input")
  ```

- Create a button to trigger the query:
  ```python
  submit = st.button("Query it")
  ```

#### **Step 8: Query Execution on Submit**
When the submit button is clicked, the application generates the SQL query using Gemini AI and retrieves the corresponding data from the SQLite database.

```python
if submit:
    response = get_gemini_response(question, prompt)
    data = read_sql_query(response, "student.db")
    st.header("The Response is")
    for row in data:
        st.json(row)
```

### How the Code Works (Overview):
1. **User Input**: The user types a natural language question, such as "How many students are in the Data Science class?"
2. **Gemini AI Processing**: The question is passed to Gemini AI, which generates the corresponding SQL query.
3. **Database Querying**: The SQL query is executed against an SQLite database.
4. **Data Display**: The retrieved data is displayed on the Streamlit web interface.