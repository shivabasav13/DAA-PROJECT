#include <iostream>
#include <string>
#include <queue>
#include <unordered_map>
#include <vector>
#include <iomanip>
#include <ctime>

using namespace std;

// Function to generate a random ID
int generateID() {
    static int id = 1000;
    return id++;
}

// Patient class definition
class Patient {
public:
    int id;
    string name;
    int emergencyLevel;
    string appointmentDetails;
    string insuranceCompany;
    double medicalExpenses;
    string staffInCharge;
    string admissionDate;
    string dischargeDate;

    Patient(int id, string name, int emergencyLevel, string appointmentDetails, string insuranceCompany, double medicalExpenses, string staffInCharge)
        : id(id), name(name), emergencyLevel(emergencyLevel), appointmentDetails(appointmentDetails), insuranceCompany(insuranceCompany), medicalExpenses(medicalExpenses), staffInCharge(staffInCharge), admissionDate("N/A"), dischargeDate("N/A") {}
};

// Resource Management
struct Resources {
    int availableAmbulances;
    int totalAmbulances;
    int availableWards;
    int totalWards;
    int parkedCars;
    int totalParkingSpots;

    Resources(int ambulances, int wards, int parkingSpots)
        : availableAmbulances(ambulances), totalAmbulances(ambulances), availableWards(wards), totalWards(wards), parkedCars(0), totalParkingSpots(parkingSpots) {}

    void displayStatus() {
        cout << "\nHospital Resource Status:\n";
        cout << "Available Ambulances: " << availableAmbulances << "/" << totalAmbulances << endl;
        cout << "Available Emergency Wards: " << availableWards << "/" << totalWards << endl;
        cout << "Parked Cars: " << parkedCars << "/" << totalParkingSpots << endl;
    }

    void updateParkingStatus(int parked) {
        parkedCars = parked;
    }

    void updateAmbulanceStatus(int available) {
        availableAmbulances = available;
    }
};

// Linked List Node
class Node {
public:
    Patient* patient;
    Node* next;

    Node(Patient* patient) : patient(patient), next(nullptr) {}
};

// Linked List Class
class LinkedList {
private:
    Node* head;

public:
    LinkedList() : head(nullptr) {}

    void addPatient(Patient* patient) {
        Node* newNode = new Node(patient);
        if (!head) {
            head = newNode;
        } else {
            Node* current = head;
            while (current->next) {
                current = current->next;
            }
            current->next = newNode;
        }
    }

    void displayPatients() {
        if (!head) {
            cout << "No patients in the system." << endl;
            return;
        }

        Node* current = head;
        while (current) {
            Patient* p = current->patient;
            cout << "ID: " << p->id << ", Name: " << p->name
                 << ", Emergency Level: " << p->emergencyLevel
                 << ", Appointment: " << p->appointmentDetails
                 << ", Insurance: " << p->insuranceCompany
                 << ", Medical Expenses: " << fixed << setprecision(2) << p->medicalExpenses
                 << ", Staff In Charge: " << p->staffInCharge
                 << ", Admission Date: " << p->admissionDate
                 << ", Discharge Date: " << p->dischargeDate << endl;
            current = current->next;
        }
    }

    bool deletePatient(int id) {
        if (!head) return false;

        if (head->patient->id == id) {
            Node* temp = head;
            head = head->next;
            delete temp;
            return true;
        }

        Node* current = head;
        while (current->next && current->next->patient->id != id) {
            current = current->next;
        }

        if (current->next) {
            Node* temp = current->next;
            current->next = current->next->next;
            delete temp;
            return true;
        }
        return false;
    }

    void updateAdmissionDate(int id, string date) {
        Node* current = head;
        while (current) {
            if (current->patient->id == id) {
                current->patient->admissionDate = date;
                return;
            }
            current = current->next;
        }
        cout << "Patient with ID " << id << " not found.\n";
    }

    void updateDischargeDate(int id, string date) {
        Node* current = head;
        while (current) {
            if (current->patient->id == id) {
                current->patient->dischargeDate = date;
                return;
            }
            current = current->next;
        }
        cout << "Patient with ID " << id << " not found.\n";
    }
};

// Priority Queue Class
class PriorityQueue {
private:
    struct Compare {
        bool operator()(Patient* a, Patient* b) {
            return a->emergencyLevel > b->emergencyLevel;
        }
    };

    priority_queue<Patient*, vector<Patient*>, Compare> queue;

public:
    void addPatient(Patient* patient) {
        queue.push(patient);
    }

    void processPatient(Resources& resources) {
        if (queue.empty()) {
            cout << "No emergency patients to process." << endl;
        } else {
            if (resources.availableWards > 0) {
                Patient* p = queue.top();
                queue.pop();
                resources.availableWards--;
                cout << "Processing Patient: " << p->name << " (ID: " << p->id << ") with Emergency Level: " << p->emergencyLevel << endl;
            } else {
                cout << "No emergency wards available for the patient." << endl;
            }
        }
    }

    void displayEmergencyPatients() {
        if (queue.empty()) {
            cout << "No emergency patients in the queue." << endl;
            return;
        }

        priority_queue<Patient*, vector<Patient*>, Compare> tempQueue = queue;
        while (!tempQueue.empty()) {
            Patient* p = tempQueue.top();
            tempQueue.pop();
            cout << "ID: " << p->id << ", Name: " << p->name << ", Emergency Level: " << p->emergencyLevel << endl;
        }
    }
};

// Hash Table Class
class HashTable {
private:
    unordered_map<int, Patient*> table;

public:
    void addPatient(Patient* patient) {
        table[patient->id] = patient;
    }

    Patient* searchById(int id) {
        if (table.find(id) != table.end()) {
            return table[id];
        } else {
            return nullptr;
        }
    }

    void searchByName(string name) {
        for (const auto& entry : table) {
            if (entry.second->name == name) {
                cout << "Patient Found: ID: " << entry.second->id << ", Name: " << entry.second->name << endl;
                return;
            }
        }
        cout << "Patient not found." << endl;
    }

    void displayInsuranceStats() {
        unordered_map<string, int> insuranceCount;
        for (const auto& entry : table) {
            insuranceCount[entry.second->insuranceCompany]++;
        }
        cout << "\nInsurance Statistics:\n";
        for (const auto& entry : insuranceCount) {
            cout << "Company: " << entry.first << ", Patients Covered: " << entry.second << endl;
        }
    }
};

// Bill Generation
void generateBill(Patient* patient) {
    double totalCost = patient->medicalExpenses;
    cout << "\n--- Medical Bill ---\n";
    cout << "Patient ID: " << patient->id << endl;
    cout << "Name: " << patient->name << endl;
    cout << "Insurance Company: " << patient->insuranceCompany << endl;
    cout << "Medical Expenses: " << fixed << setprecision(2) << totalCost << endl;
    cout << "----------------------\n";
}

// Ambulance Management
void manageAmbulance(Resources& resources) {
    int action, num;
    cout << "1. Add Ambulance\n2. Remove Ambulance\nEnter your choice: ";
    cin >> action;
    if (action == 1) {
        cout << "Enter number of ambulances to add: ";
        cin >> num;
        resources.totalAmbulances += num;
        resources.availableAmbulances += num;
    } else if (action == 2) {
        cout << "Enter number of ambulances to remove: ";
        cin >> num;
        if (num <= resources.availableAmbulances) {
            resources.totalAmbulances -= num;
            resources.availableAmbulances -= num;
        } else {
            cout << "Not enough available ambulances to remove.\n";
        }
    } else {
        cout << "Invalid choice.\n";
    }
}

// Parking Management
void manageParking(Resources& resources) {
    int action, num;
    cout << "1. Park a Car\n2. Remove a Car\nEnter your choice: ";
    cin >> action;
    if (action == 1) {
        if (resources.parkedCars < resources.totalParkingSpots) {
            resources.parkedCars++;
            cout << "Car parked successfully.\n";
        } else {
            cout << "Parking lot is full.\n";
        }
    } else if (action == 2) {
        if (resources.parkedCars > 0) {
            resources.parkedCars--;
            cout << "Car removed successfully.\n";
        } else {
            cout << "Parking lot is empty.\n";
        }
    } else {
        cout << "Invalid choice.\n";
    }
}

// Staff Management
void assignStaffToPatient(Patient* patient, const string& staffName) {
    patient->staffInCharge = staffName;
    cout << "Staff " << staffName << " assigned to patient " << patient->name << " (ID: " << patient->id << ").\n";
}

// Admission and Discharge Dates Management
void updatePatientAdmissionDate(LinkedList& patientList, int id, const string& date) {
    patientList.updateAdmissionDate(id, date);
    cout << "Admission date updated successfully.\n";
}

void updatePatientDischargeDate(LinkedList& patientList, int id, const string& date) {
    patientList.updateDischargeDate(id, date);
    cout << "Discharge date updated successfully.\n";
}

// Main Function
int main() {
    LinkedList patientList;
    PriorityQueue emergencyQueue;
    HashTable patientTable;
    Resources hospitalResources(5, 10, 50);  // Example: 5 ambulances, 10 emergency wards, 50 parking spots

    int choice;
    do {
        cout << "\nHospital Management System Menu:\n";
        cout << "1. Add Patient\n";
        cout << "2. Display All Patients\n";
        cout << "3. Search Patient by ID\n";
        cout << "4. Search Patient by Name\n";
        cout << "5. Process Emergency Patient\n";
        cout << "6. Display Emergency Patients\n";
        cout << "7. Display Resource Status\n";
        cout << "8. Display Insurance Statistics\n";
        cout << "9. Update Parking Status\n";
        cout << "10. Update Ambulance Status\n";
        cout << "11. Generate Medical Bill\n";
        cout << "12. Assign Staff to Patient\n";
        cout << "13. Update Patient Admission Date\n";
        cout << "14. Update Patient Discharge Date\n";
        cout << "15. Exit\n";
        cout << "Enter your choice: ";
        cin >> choice;

        switch (choice) {
        case 1: {
            int emergencyLevel;
            string name, appointmentDetails, insuranceCompany, staffInCharge;
            double medicalExpenses;
            cout << "Enter Name: ";
            cin.ignore();
            getline(cin, name);
            cout << "Enter Emergency Level (1-5): ";
            cin >> emergencyLevel;
            cin.ignore();
            cout << "Enter Appointment Details: ";
            getline(cin, appointmentDetails);
            cout << "Enter Insurance Company: ";
            getline(cin, insuranceCompany);
            cout << "Enter Medical Expenses: ";
            cin >> medicalExpenses;
            cin.ignore();
            cout << "Enter Staff In Charge: ";
            getline(cin, staffInCharge);

            int id = generateID();
            Patient* newPatient = new Patient(id, name, emergencyLevel, appointmentDetails, insuranceCompany, medicalExpenses, staffInCharge);
            patientList.addPatient(newPatient);
            patientTable.addPatient(newPatient);
            if (emergencyLevel <= 2) {
                emergencyQueue.addPatient(newPatient);
            }
            cout << "Patient added successfully.\n";
            break;
        }
        case 2:
            patientList.displayPatients();
            break;
        case 3: {
            int id;
            cout << "Enter Patient ID: ";
            cin >> id;
            Patient* patient = patientTable.searchById(id);
            if (patient) {
                cout << "Patient Found: ID: " << patient->id << ", Name: " << patient->name
                     << ", Emergency Level: " << patient->emergencyLevel
                     << ", Appointment: " << patient->appointmentDetails
                     << ", Insurance: " << patient->insuranceCompany
                     << ", Medical Expenses: " << fixed << setprecision(2) << patient->medicalExpenses
                     << ", Staff In Charge: " << patient->staffInCharge
                     << ", Admission Date: " << patient->admissionDate
                     << ", Discharge Date: " << patient->dischargeDate << endl;
            } else {
                cout << "Patient not found.\n";
            }
            break;
        }
        case 4: {
            string name;
            cout << "Enter Patient Name: ";
            cin.ignore();
            getline(cin, name);
            patientTable.searchByName(name);
            break;
        }
        case 5:
            emergencyQueue.processPatient(hospitalResources);
            break;
        case 6:
            emergencyQueue.displayEmergencyPatients();
            break;
        case 7:
            hospitalResources.displayStatus();
            break;
        case 8:
            patientTable.displayInsuranceStats();
            break;
        case 9: {
            int parked;
            cout << "Enter number of parked cars: ";
            cin >> parked;
            hospitalResources.updateParkingStatus(parked);
            break;
        }
        case 10: {
            int available;
            cout << "Enter number of available ambulances: ";
            cin >> available;
            hospitalResources.updateAmbulanceStatus(available);
            break;
        }
        case 11: {
            int id;
            cout << "Enter Patient ID for bill generation: ";
            cin >> id;
            Patient* patient = patientTable.searchById(id);
            if (patient) {
                generateBill(patient);
            } else {
                cout << "Patient not found.\n";
            }
            break;
        }
        case 12: {
            int id;
            string staffName;
            cout << "Enter Patient ID: ";
            cin >> id;
            cin.ignore();
            cout << "Enter Staff Name: ";
            getline(cin, staffName);
            Patient* patient = patientTable.searchById(id);
            if (patient) {
                assignStaffToPatient(patient, staffName);
            } else {
                cout << "Patient not found.\n";
            }
            break;
        }
        case 13: {
            int id;
            string date;
            cout << "Enter Patient ID: ";
            cin >> id;
            cin.ignore();
            cout << "Enter Admission Date (YYYY-MM-DD): ";
            getline(cin, date);
            updatePatientAdmissionDate(patientList, id, date);
            break;
        }
        case 14: {
            int id;
            string date;
            cout << "Enter Patient ID: ";
            cin >> id;
            cin.ignore();
            cout << "Enter Discharge Date (YYYY-MM-DD): ";
            getline(cin, date);
            updatePatientDischargeDate(patientList, id, date);
            break;
        }
        case 15:
            cout << "Exiting...\n";
            break;
        default:
            cout << "Invalid choice. Please try again.\n";
            break;
        }
    } while (choice != 15);

    return 0;
}
