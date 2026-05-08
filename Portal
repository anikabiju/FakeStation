import streamlit as st
import pandas as pd
import random

st.set_page_config(page_title="FakeStation Portal", layout="wide")
st.title("🔬 FakeStation Portal")

# 1. ADMIN SETUP
st.sidebar.header("Admin Configuration")
uploaded_file = st.sidebar.file_uploader("Upload Well Layout (Excel)", type=["xlsx"])
if uploaded_file:
    df = pd.read_excel(uploaded_file)
    df['Sample_Name'] = df['Sample_Name'].astype(str)
    df['Well_ID'] = df['Well_ID'].astype(str)
    
    valid_targets = [
        x for x in df['Target'].unique() 
        if pd.notna(x) and str(x).strip().upper() != "N/A"
    ]
    
    # Convert everything to a clean string and sort (A, B, C...)
    possible_choices = sorted([str(x).strip() for x in valid_targets])

    # Initialize session states
    if 'random_order' not in st.session_state:
        indices = df.index.tolist()
        random.shuffle(indices)
        st.session_state.random_order = indices
        st.session_state.current_step = 0
        st.session_state.results = []
        st.session_state.waiting_for_next = False
        st.session_state.last_feedback = None

    # 2. PANELIST INTERFACE
    step = st.session_state.current_step
    if step < len(st.session_state.random_order):
        idx = st.session_state.random_order[step]
        current_well = df.iloc[idx]

        st.subheader(f"Sample {step + 1} of {len(df)}")
        st.info(f"Please taste the sample in Well: **{current_well['Well_ID']}**")

        # Input Section using the dynamic choices found in your Excel file
        choice = st.radio(
            "Select Response:", 
            possible_choices, 
            index=None, 
            disabled=st.session_state.waiting_for_next,
            help="Select the coordinate or target that matches this sample."
        )

        # Logic for Submit vs Next
        if not st.session_state.waiting_for_next:
            if st.button("Submit Response", disabled=(choice is None)):
                is_correct = "N/A"
                
                # Logic for Control Samples
                if current_well['Type'].strip().lower() == 'control':
                    is_correct = (str(choice) == str(current_well['Target']))
                    if is_correct:
                        st.session_state.last_feedback = f"Correct! ✅ The sample was indeed {current_well['Target']}."
                    else:
                        st.session_state.last_feedback = f"Incorrect. ❌ The target was {current_well['Target']}."
                else:
                    st.session_state.last_feedback = "Response Recorded."

                # Save detailed results
                st.session_state.results.append({
                    "Well_ID": current_well['Well_ID'],
                    "Sample_Name": current_well['Sample_Name'],
                    "Type": current_well['Type'],
                    "Target": current_well['Target'],
                    "User_Choice": choice,
                    "Is_Correct": is_correct
                })
                
                st.session_state.waiting_for_next = True
                st.rerun()

        else:
            # Display Feedback
            if "Correct!" in st.session_state.last_feedback:
                st.success(st.session_state.last_feedback)
            elif "Incorrect" in st.session_state.last_feedback:
                st.error(st.session_state.last_feedback)
            else:
                st.info(st.session_state.last_feedback)

            if st.button("Proceed to Next Sample ➡️"):
                st.session_state.current_step += 1
                st.session_state.waiting_for_next = False
                st.session_state.last_feedback = None
                st.rerun()

    else:
        st.success("Testing Complete!")
        
        # 3. ADVANCED DYNAMIC REPORTING
        results_df = pd.DataFrame(st.session_state.results)
        st.header("📊 Final Session Report")
        
        if not results_df.empty:
            # Create a pivot table showing percentages for whatever choices were available
            report = pd.crosstab(
                results_df['Sample_Name'], 
                results_df['User_Choice'], 
                normalize='index'
            ) * 100
            
            # Ensure all possible choices are represented as columns even if never selected
            for col in possible_choices:
                if col not in report.columns:
                    report[col] = 0.0
            
            # Sort columns A, B, C...
            report = report[sorted(report.columns)]
            
            st.subheader("Selection Proportions per Sample")
            st.table(report.style.format("{:.1f}%"))
            
            st.subheader("Visual Distribution")
            st.bar_chart(report)

        if st.button("Restart Session"):
            for key in list(st.session_state.keys()):
                del st.session_state[key]
            st.rerun()
else:
    st.warning("Please upload your well layout file to begin.")
