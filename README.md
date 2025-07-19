# CAALWP2
import streamlit as st
import pandas as pd
from datetime import datetime

def generate_medi_cal_270(row, batch=True):
    now = datetime.now()
    date = now.strftime("%y%m%d")
    time = now.strftime("%H%M")
    long_date = now.strftime("%Y%m%d")
    ctrl_num = "000000001"
    st_ctrl_num = "0001"
    submitter_id = row['SubmitterID']
    provider_number = row['ProviderNumber']
    subscriber_id = row['SubscriberID']
    patient_last = row['PatientLast']
    patient_first = row['PatientFirst']
    patient_dob = row['PatientDOB']
    patient_gender = row['PatientGender']

    edi = [
        f"ISA*00*          *01*          *ZZ*{submitter_id:<15}*ZZ*610442        *{date}*{time}*^*00501*{ctrl_num}*1*T*:~",
        f"GS*HS*{submitter_id}*610442*{long_date}*{time}*1*X*005010X279A1~",
        f"ST*270*{st_ctrl_num}*005010X279A1~",
        f"BHT*0022*13*10001234*{long_date}*{time}~",
        f"HL*1**20*1~",
        f"NM1*PR*2*Medi-Cal*****PI*610442~",
        f"HL*2*1*21*1~",
        f"NM1*1P*2*Your Clinic*****XX*{provider_number}~",
        f"REF*4A*{submitter_id}~",
        f"HL*3*2*22*0~",
        f"NM1*IL*1*{patient_last}*{patient_first}****MI*{subscriber_id}~",
        f"DMG*D8*{patient_dob}*{patient_gender}~",
        f"DTP*291*D8*{long_date}~",
        f"EQ*30~",
        f"SE*13*{st_ctrl_num}~",
        f"GE*1*1~",
        f"IEA*1*{ctrl_num}~"
    ]
    return '\n'.join(edi)

# --- Streamlit App ---
st.title("Medi-Cal EDI File Generator")

uploaded_file = st.file_uploader("Upload Billing Template (Excel)", type=["xlsx"])

if uploaded_file:
    df = pd.read_excel(uploaded_file)
    st.write("Billing Template Preview:", df.head())

    if not df.empty:
        row_idx = st.number_input(
            "Select row number to generate EDI (starting at 0):",
            min_value=0, max_value=len(df)-1, value=0, step=1
        )
        selected_row = df.iloc[row_idx]
        st.write("Selected Record:", selected_row)
        
        if st.button("Generate EDI 270 File"):
            edi_text = generate_medi_cal_270(selected_row)
            st.download_button(
                label="Download EDI 270 File",
                data=edi_text,
                file_name="medi_cal_270.edi",
                mime="text/plain"
            )
