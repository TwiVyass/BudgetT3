# BudgetT3
This is a very basic Streamlit webpage designed to work as a Budget Tracker for college students where you can remove and add items applicable to the student/user




import pandas as pd
import plotly.express as px
import streamlit as st
import calendar

st.title("College Budget")

# Load the data
df = pd.read_excel(
    io='College budget1.xlsx',
    sheet_name='Monthly Living Expenses',
    engine='openpyxl',
    skiprows=4,
    usecols='B:C',
    nrows=13,
)

year = st.sidebar.number_input("Enter a year:", value=2023, min_value=1, max_value=9999, step=1)
month = st.sidebar.selectbox("Select a month:", options=range(1, 13))

# Display the calendar only on the sidebar
cal = calendar.month(year, month, 2, 3)
st.sidebar.write(f"Calendar for {year}-{month}")
st.sidebar.text(cal)

st.title("Budget Tracker")

print("Check whichever is applicable:")

if st.checkbox("Do you have an allowance?"):
    allowance = st.number_input("Enter your allowance:", min_value=0.0, step=0.01)
    st.write(f"Your allowance is {allowance}")
else:
    allowance = 0.0
    
if st.checkbox("Do you have a salary?"):
    salary=st.number_input("Enter your salary:", min_value=0.0, step=0.01)
    st.write(f"Your salary is {salary} ")
else:
    salary=0.0
    

# Create the sidebar for selecting columns to display
if len(df.columns) > 0:
    cols_to_display_options = df.columns
    cols_to_display = st.sidebar.multiselect(
        "Columns:",
        options=["Item","Amount"],
        default=["Item", "Amount"]
    )
    if not set(cols_to_display).issubset(cols_to_display_options):
        st.sidebar.warning("At least one default column does not exist in the dataframe.")
else:
    st.sidebar.warning("The dataframe doesn't have any columns to display.")
    
    


# Create the sidebar for selecting items to update
st.sidebar.header("Deselect items that are not applicable:")
items_to_update = st.sidebar.multiselect(
    "Items:",
    options=df["Item"].unique(),
    default=df["Item"].unique()
)


# Allow the user to enter new values for selected items and columns
for item in items_to_update:
    st.sidebar.write(f"Enter new values for {item}:")
    item_row = df.loc[df['Item'] == item]
    for col in cols_to_display:
        if col != "Item":
            old_value = float(item_row[col])
            new_value = st.sidebar.number_input(
                f"{col}:",
                key=f"{item}-{col}",  # add a unique key based on item and col
                value=old_value,
                step=0.01
            )
            if new_value != old_value:
                df.at[item_row.index[0], col] = new_value

# Display the updated table with the selected columns and items
df_filtered = df.loc[df["Item"].isin(items_to_update), cols_to_display]
st.dataframe(df_filtered)

total_amount = df_filtered['Amount'].sum()


# Calculate the remaining budget
if allowance > 0.0:
    remaining_budget = allowance - total_amount
    st.write(f"Remaining budget: {remaining_budget}")
elif salary > 0.0:
    remaining_budget = salary - total_amount
    st.write(f"Remaining budget: {remaining_budget}")

    
# Display the calendar only on the sidebar
st.sidebar.write(f"Calendar for {year}-{month}")
calendar_days = calendar.monthcalendar(year, month)
for week in calendar_days:
    for day in week:
        if day != 0:
            amount_spent = st.sidebar.number_input(f"Amount spent on {day}/{month}/{year}:", value=0.0, step=0.01)
            st.sidebar.write(f"You spent {amount_spent} on {day}/{month}/{year}")
            
# Create a bar chart to visualize the data
fig = px.bar(df_filtered, x="Item", y="Amount")
st.plotly_chart(fig)


