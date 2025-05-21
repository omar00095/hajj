import streamlit as st
import pandas as pd

st.title("📊 نظام توزيع الكوادر الطبية بين المستشفيات")

default_data = {
    "Hospital Name": [
        "منى 1", "منى 2", "منى 3", "منى 4", "منى 5", "منى 6", "منى 7",
        "منى 8", "منى 9", "منى 10", "منى 11", "منى 12", "منى 13", "منى 14",
        "عرفة 1", "عرفة 2", "عرفة 3", "عرفة 4", "عرفة 5", "عرفة 6", "عرفة 7"
    ],
    "Capacity": [5, 6, 4, 6, 5, 4, 3, 5, 5, 3, 4, 6, 5, 5, 4, 3, 5, 4, 4, 3, 3],
    "Current Physicians": [2, 5, 4, 5, 4, 2, 2, 6, 5, 3, 2, 5, 3, 3, 2, 2, 4, 2, 2, 2, 2],
    "Visits in 30 Min": [20, 10, 11, 14, 20, 17, 15, 5, 12, 4, 10, 7, 20, 8, 7, 11, 18, 4, 12, 2, 6]
}
df = pd.DataFrame(default_data)

st.subheader("🔧 أدخل البيانات الخاصة بالمستشفيات")
editable_df = st.data_editor(df, num_rows="dynamic")

def hospital_staff_movement(df):
    df["Load"] = df["Visits in 30 Min"] / df["Current Physicians"]

    def classify_status(load):
        if load > 4:
            return "Overloaded"
        elif load < 2:
            return "Overstaffed"
        else:
            return "Optimal"
    
    df["Status"] = df["Load"].apply(classify_status)
    df["Required Physicians"] = (df["Visits in 30 Min"] / 4).round().astype(int)

    df["Staff to Move"] = df.apply(
        lambda row: row["Required Physicians"] - row["Current Physicians"] if row["Status"] == "Overloaded" else None,
        axis=1
    )

    overloaded = df[df["Status"] == "Overloaded"].copy()
    overstaffed = df[df["Status"] == "Overstaffed"].copy()

    movements = []
    for _, needy in overloaded.iterrows():
        staff_needed = needy["Staff to Move"]
        if staff_needed <= 0:
            continue

        for idx, source in overstaffed.iterrows():
            surplus = source["Current Physicians"] - source["Required Physicians"]
            if surplus <= 0:
                continue

            move = min(surplus, staff_needed)
            if move > 0:
                movements.append({
                    "From Hospital": source["Hospital Name"],
                    "To Hospital": needy["Hospital Name"],
                    "Staff to Move": move
                })
                overstaffed.at[idx, "Current Physicians"] -= move
                staff_needed -= move

            if staff_needed <= 0:
                break

    movements_df = pd.DataFrame(movements)
    return df, movements_df

status_df, movement_df = hospital_staff_movement(editable_df)

st.subheader("📋 حالة كل مستشفى")
st.dataframe(status_df[["Hospital Name", "Current Physicians", "Visits in 30 Min", "Load", "Status", "Required Physicians", "Staff to Move"]])

st.subheader("🚑 اقتراحات توزيع الكوادر الطبية")
if not movement_df.empty:
    st.dataframe(movement_df)
else:
    st.info("لا توجد حركات مطلوبة حالياً.")
