import json
import os

# ================== HOSPITAL MANAGEMENT SYSTEM ==================

DATA_FILE = "patients.json"
patients = {}

# -------------------- HELPER FUNCTIONS --------------------
def load_data():
    """Load patient data from JSON file."""
    global patients
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r") as f:
            patients = json.load(f)
    else:
        patients = {}

def save_data():
    """Save patient data to JSON file."""
    with open(DATA_FILE, "w") as f:
        json.dump(patients, f, indent=4)

def calculate_total_bill(patient_id):
    """Calculate total bill for a patient."""
    return sum(patients[patient_id]["treatments"].values())

def display_separator():
    print("-" * 60)

def get_positive_number(prompt):
    """Get a positive number from user input."""
    while True:
        try:
            value = float(input(prompt))
            if value <= 0:
                print("  Value must be positive.")
            else:
                return value
        except ValueError:
            print("  Invalid input. Enter a number.")

# -------------------- 1. ADD PATIENT --------------------
def add_patient():
    print("\n===== ADD PATIENT =====")
    patient_id = input("Enter Patient ID: ").strip()
    if patient_id in patients:
        print(f"Error: Patient ID '{patient_id}' already exists.")
        return

    full_name = input("Enter Full Name: ").strip()
    
    while True:
        try:
            age = int(input("Enter Age: "))
            if age <= 0:
                print("Age must be a positive number.")
            else:
                break
        except ValueError:
            print("Invalid input. Please enter a number.")

    gender = input("Enter Gender (Male/Female/Other): ").strip()
    diagnosis = input("Enter Diagnosis: ").strip()

    treatments = {}
    print("Enter at least 2 treatments with costs:")
    treatment_count = 0
    while True:
        treatment_name = input(f"  Treatment {treatment_count + 1} name (or 'done'): ").strip()
        if treatment_name.lower() == "done":
            if treatment_count < 2:
                print("  You must enter at least 2 treatments.")
                continue
            else:
                break
        cost = get_positive_number(f"  Cost for '{treatment_name}': ")
        treatments[treatment_name] = cost
        treatment_count += 1

    patients[patient_id] = {
        "name": full_name,
        "age": age,
        "gender": gender,
        "diagnosis": diagnosis,
        "treatments": treatments
    }

    save_data()
    print(f"\nPatient '{full_name}' added successfully!")

# -------------------- 2. VIEW ALL PATIENTS --------------------
def view_all_patients():
    print("\n===== ALL PATIENTS =====")
    if not patients:
        print("No patients registered yet.")
        return
    display_separator()
    print(f"{'Patient ID':<12} {'Name':<25} {'Diagnosis':<20}")
    display_separator()
    for pid, info in patients.items():
        print(f"{pid:<12} {info['name']:<25} {info['diagnosis']:<20}")
    display_separator()

# -------------------- 3. VIEW PATIENT REPORT --------------------
def view_patient_report():
    print("\n===== VIEW PATIENT REPORT =====")
    patient_id = input("Enter Patient ID: ").strip()
    if patient_id not in patients:
        print(f"Error: Patient ID '{patient_id}' not found.")
        return
    p = patients[patient_id]
    display_separator()
    print(f"Patient ID : {patient_id}")
    print(f"Name       : {p['name']}")
    print(f"Age        : {p['age']}")
    print(f"Gender     : {p['gender']}")
    print(f"Diagnosis  : {p['diagnosis']}")
    print("\nTreatments & Costs:")
    for t, c in p["treatments"].items():
        print(f"  - {t:<25} K{c:.2f}")
    display_separator()
    print(f"TOTAL BILL : K{calculate_total_bill(patient_id):.2f}")
    display_separator()

# -------------------- 4. UPDATE PATIENT --------------------
def update_patient():
    print("\n===== UPDATE PATIENT =====")
    patient_id = input("Enter Patient ID to update: ").strip()
    if patient_id not in patients:
        print(f"Error: Patient ID '{patient_id}' not found.")
        return
    p = patients[patient_id]

    print("\nUpdating patient:", p["name"])
    print("a. Update Diagnosis")
    print("b. Add New Treatment")
    print("c. Update Treatment Cost")
    print("d. Remove Treatment")
    choice = input("Select option (a/b/c/d): ").strip().lower()

    if choice == "a":
        p["diagnosis"] = input("Enter new diagnosis: ").strip()
        print("Diagnosis updated.")
    elif choice == "b":
        treatment_name = input("Enter new treatment name: ").strip()
        cost = get_positive_number(f"Enter cost for '{treatment_name}': ")
        p["treatments"][treatment_name] = cost
        print("Treatment added.")
    elif choice == "c":
        if not p["treatments"]:
            print("No treatments to update.")
            return
        print("Current treatments:", list(p["treatments"].keys()))
        t_name = input("Enter treatment name to update: ").strip()
        if t_name not in p["treatments"]:
            print("Treatment not found.")
            return
        new_cost = get_positive_number(f"Enter new cost for '{t_name}': ")
        p["treatments"][t_name] = new_cost
        print("Treatment cost updated.")
    elif choice == "d":
        if not p["treatments"]:
            print("No treatments to remove.")
            return
        print("Current treatments:", list(p["treatments"].keys()))
        t_name = input("Enter treatment name to remove: ").strip()
        if t_name not in p["treatments"]:
            print("Treatment not found.")
            return
        del p["treatments"][t_name]
        print("Treatment removed.")
    else:
        print("Invalid option.")

    save_data()

# -------------------- 5. DELETE PATIENT --------------------
def delete_patient():
    print("\n===== DELETE PATIENT =====")
    patient_id = input("Enter Patient ID to delete: ").strip()
    if patient_id not in patients:
        print("Patient not found.")
        return
    confirm = input(f"Confirm deletion of '{patients[patient_id]['name']}'? (yes/no): ").strip().lower()
    if confirm == "yes":
        del patients[patient_id]
        save_data()
        print("Patient deleted.")
    else:
        print("Deletion cancelled.")

# -------------------- 6. SEARCH PATIENT --------------------
def search_patient():
    print("\n===== SEARCH PATIENT =====")
    term = input("Enter Patient ID or Name: ").strip().lower()
    results = [(pid, info) for pid, info in patients.items()
               if term == pid.lower() or term in info["name"].lower()]
    if not results:
        print("No matching patients found.")
        return
    display_separator()
    print(f"{'Patient ID':<12} {'Name':<25} {'Diagnosis':<20}")
    display_separator()
    for pid, info in results:
        print(f"{pid:<12} {info['name']:<25} {info['diagnosis']:<20}")
    display_separator()

# -------------------- 7. HOSPITAL STATISTICS --------------------
def hospital_statistics():
    print("\n===== HOSPITAL STATISTICS =====")
    if not patients:
        print("No patients registered yet.")
        return

    total_patients = len(patients)
    bills = {pid: calculate_total_bill(pid) for pid in patients}
    total_revenue = sum(bills.values())
    highest_id = max(bills, key=bills.get)
    lowest_id = min(bills, key=bills.get)

    display_separator()
    print(f"Total Patients          : {total_patients}")
    print(f"Total Revenue           : K{total_revenue:.2f}")
    print(f"Highest Bill Patient    : {patients[highest_id]['name']} (K{bills[highest_id]:.2f})")
    print(f"Lowest Bill Patient     : {patients[lowest_id]['name']} (K{bills[lowest_id]:.2f})")
    display_separator()

# -------------------- MAIN MENU --------------------
def display_menu():
    print("\n" + "=" * 50)
    print("       HOSPITAL MANAGEMENT SYSTEM       ")
    print("=" * 50)
    print("1. Add Patient")
    print("2. View All Patients")
    print("3. View Patient Report")
    print("4. Update Patient")
    print("5. Delete Patient")
    print("6. Search Patient")
    print("7. Hospital Statistics")
    print("8. Exit")
    print("=" * 50)

def main():
    load_data()
    while True:
        display_menu()
        choice = input("Enter choice (1-8): ").strip()
        if choice == "1":
            add_patient()
        elif choice == "2":
            view_all_patients()
        elif choice == "3":
            view_patient_report()
        elif choice == "4":
            update_patient()
        elif choice == "5":
            delete_patient()
        elif choice == "6":
            search_patient()
        elif choice == "7":
            hospital_statistics()
        elif choice == "8":
            print("\nThank you for using the Hospital Management System. Goodbye!")
            break
        else:
            print("Invalid choice. Enter 1-8.")

# -------------------- ENTRY POINT --------------------
if __name__ == "__main__":
    main()