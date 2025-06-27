import streamlit as st

st.title("Student Performance Analyzer")
menu = st.sidebar.selectbox("Menu", ["Enter Data", "View Reports", "Graphs"])
import sqlite3

conn = sqlite3.connect("students.db")
c = conn.cursor()
c.execute('''CREATE TABLE IF NOT EXISTS performance
             (roll TEXT, name TEXT, subject TEXT, marks INTEGER, attendance INTEGER)''')
conn.commit()
if menu == "Enter Data":
    name = st.text_input("Name")
    roll = st.text_input("Roll Number")
    subject = st.selectbox("Subject", ["Math", "Science", "English"])
    marks = st.slider("Marks", 0, 100)
    attendance = st.slider("Attendance (%)", 0, 100)
    
    if st.button("Submit"):
        c.execute("INSERT INTO performance VALUES (?, ?, ?, ?, ?)", (roll, name, subject, marks, attendance))
        conn.commit()
        st.success("Data submitted")
import pandas as pd
import plotly.express as px

if menu == "Graphs":
    df = pd.read_sql_query("SELECT * FROM performance", conn)
    fig = px.bar(df, x='subject', y='marks', color='name', barmode='group')
    st.plotly_chart(fig)
from fpdf import FPDF

if st.button("Generate PDF Report"):
    pdf = FPDF()
    pdf.add_page()
    pdf.set_font("Arial", size=12)
    for i, row in df.iterrows():
        pdf.cell(200, 10, txt=f"{row['name']} - {row['subject']} - {row['marks']} marks", ln=True)
    pdf.output("student_report.pdf")
    st.success("Report generated!")
