import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import math

# --- Page setup ---
st.set_page_config(page_title="Mortgage Repayments Calculator", page_icon="🏡", layout="centered")

# --- Custom background and styling ---
st.markdown(
    """
    <style>
    body {
        background-color: #f8f9fa;
        color: #333333;
    }
    .stApp {
        background-color: #f8f9fa;
    }
    h1, h2, h3 {
        color: #004080;
    }
    .explanation-box {
        background-color: #e8f0fe;
        border-left: 6px solid #1f77b4;
        padding: 15px;
        border-radius: 5px;
        margin-bottom: 20px;
    }
    </style>
    """,
    unsafe_allow_html=True
)

# --- Title ---
st.title("🏡 Mortgage Repayments Calculator")

st.markdown(
    """
    <div class="explanation-box">
        This tool helps you estimate your **monthly mortgage repayments**, total interest,  
        and how your loan balance decreases over time.  
        Enter your details below, then click **Calculate** to view your personalized results.
    </div>
    """,
    unsafe_allow_html=True
)

# --- User Input ---
st.write("### ✏️ Enter Your Loan Details")
col1, col2 = st.columns(2)
home_value = col1.number_input("🏠 Home Value ($)", min_value=0, value=500000, step=10000)
deposit = col1.number_input("💰 Deposit ($)", min_value=0, value=100000, step=5000)
interest_rate = col2.number_input("📈 Interest Rate (in %)", min_value=0.0, value=5.5, step=0.1)
loan_term = col2.number_input("📅 Loan Term (in years)", min_value=1, value=30)

# --- Calculate Button ---
calculate = st.button("📊 Calculate Repayments", use_container_width=True)

# --- Only run below when button is clicked ---
if calculate:
    # --- Calculations ---
    loan_amount = home_value - deposit
    monthly_interest_rate = (interest_rate / 100) / 12
    number_of_payments = loan_term * 12

    if monthly_interest_rate == 0:
        monthly_payment = loan_amount / number_of_payments
    else:
        monthly_payment = (
            loan_amount
            * (monthly_interest_rate * (1 + monthly_interest_rate) ** number_of_payments)
            / ((1 + monthly_interest_rate) ** number_of_payments - 1)
        )

    total_payments = monthly_payment * number_of_payments
    total_interest = total_payments - loan_amount

    # --- Repayment Summary ---
    st.write("### 💰 Your Repayment Summary")
    st.markdown(
        """
        <div class="explanation-box">
            Here's a quick summary of your mortgage results.  
            - **Monthly Repayment:** Your regular payment amount.  
            - **Total Repayments:** The full amount you'll pay over the life of the loan.  
            - **Total Interest:** The total interest paid on top of the loan amount.
        </div>
        """,
        unsafe_allow_html=True
    )

    col1, col2, col3 = st.columns(3)
    col1.metric(label="Monthly Repayment", value=f"${monthly_payment:,.2f}")
    col2.metric(label="Total Repayments", value=f"${total_payments:,.0f}")
    col3.metric(label="Total Interest", value=f"${total_interest:,.0f}")

    # --- Payment Schedule ---
    schedule = []
    remaining_balance = loan_amount

    for i in range(1, number_of_payments + 1):
        interest_payment = remaining_balance * monthly_interest_rate
        principal_payment = monthly_payment - interest_payment
        remaining_balance -= principal_payment
        year = math.ceil(i / 12)
        schedule.append([i, monthly_payment, principal_payment, interest_payment, remaining_balance, year])

    df = pd.DataFrame(
        schedule,
        columns=["Month", "Payment", "Principal", "Interest", "Remaining Balance", "Year"],
    )

    st.write("### 📉 Payment Schedule and Visuals")
    st.markdown(
        """
        <div class="explanation-box">
            The charts below show how your **remaining balance** decreases over time  
            and how much of each year's payment goes toward **principal vs interest**.
        </div>
        """,
        unsafe_allow_html=True
    )

    # --- Chart 1: Remaining Balance ---
    payments_df = df[["Year", "Remaining Balance"]].groupby("Year").min()

    fig, ax = plt.subplots(figsize=(8, 5))
    ax.plot(
        payments_df.index,
        payments_df["Remaining Balance"],
        color="#1f77b4",
        linewidth=2.5,
        marker="o",
        markerfacecolor="#ff7f0e",
    )
    ax.fill_between(
        payments_df.index,
        payments_df["Remaining Balance"],
        color="#1f77b4",
        alpha=0.2,
    )
    ax.set_title("Loan Balance Over Time", fontsize=14, color="#004080")
    ax.set_xlabel("Year", fontsize=12)
    ax.set_ylabel("Remaining Balance ($)", fontsize=12)
    ax.grid(True, linestyle="--", alpha=0.6)
    st.pyplot(fig)

    # --- Chart 2: Interest vs Principal ---
    yearly_summary = df.groupby("Year")[["Principal", "Interest"]].sum()

    fig2, ax2 = plt.subplots(figsize=(8, 5))
    ax2.bar(
        yearly_summary.index,
        yearly_summary["Principal"],
        color="#2ca02c",
        label="Principal",
    )
    ax2.bar(
        yearly_summary.index,
        yearly_summary["Interest"],
        bottom=yearly_summary["Principal"],
        color="#d62728",
        label="Interest",
    )
    ax2.set_title("Yearly Payment Breakdown", fontsize=14, color="#004080")
    ax2.set_xlabel("Year", fontsize=12)
    ax2.set_ylabel("Amount ($)", fontsize=12)
    ax2.legend()
    ax2.grid(True, linestyle="--", alpha=0.6)
    st.pyplot(fig2)

    # --- Final note ---
    st.markdown(
        """
        <div class="explanation-box">
            ✅ **Tip:** Try adjusting the interest rate or deposit amount  
            to see how it changes your repayments and total interest.  
            A small change can save you thousands over time!
        </div>
        """,
        unsafe_allow_html=True
    )
else:
    st.info("👆 Enter your loan details and click **Calculate Repayments** to view results.")
