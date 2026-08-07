# hospital-management-system-
A C++ console application for hospital management featuring patient registration, doctor scheduling, patient admission/discharge tracking, automatic doctor assignment based on day/timing/gender preference, appointment history logging, and persistent data storage using file handling.
#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include <cctype>
#include <ctime>
#include<windows.h>
#include<conio.h>
#include<cstdio>
using namespace std;
void color(int c )
{
	SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE),c);
}
//getpassword
string loginpassword()
{

	string password = "";
	
	char pw;
	
	while ((pw= _getch())!='\r')
	{

		if(pw=='\b')
		{
			if(!password.empty())
			{
				password.erase(password.length()-1);
				cout<<"\b\b";
			}
		}
		else
	{
		password +=pw;
		cout<<"*";
	}
 }
cout<<"\n";
     
return password;
}

// ===== VALIDATION HELPERS =====

bool isValidName(string s)
{
    if(s.empty()) return false;
    for(char c : s)
    {
        if(!isalpha(c) && c != ' ')
            return false;
    }
    return true;
}

bool isValidAge(string s)
{
    if(s.empty()) return false;
    for(char c : s)
    {
        if(!isdigit(c))
            return false;
    }
    int age = stoi(s);
    return (age > 0 && age < 150);
}

//================ PERSON CLASS =================

class Person
{
protected:

string name;
int age;
string cnic;
string phoneNo;
string gender;

public:

Person()
{
    name = "";
    age = 0;
    cnic = "a";
    phoneNo = "";
    gender = "";
}

bool validCNIC(string cnic)
{
    if(cnic.length() != 13)
        return false;
    for(char ch : cnic)
    {
        if(!isdigit(ch))
            return false;
    }

    return true;
}

bool validPhone(string phone)
{
    if(phone.length() != 11)
        return false;

    for(char ch : phone)
    {
        if(!isdigit(ch))
            return false;
    }

    return true;
}

virtual void inputPerson()
{
    cin.ignore();

    // NAME VALIDATION
    do
    {
        cout << "Enter Name: ";
        getline(cin, name);
        if(!isValidName(name))
            cout << "Invalid Name! Name sirf alphabets aur spaces honi chahiye.\n";
    } while(!isValidName(name));

    // AGE VALIDATION
    string ageStr;
    do
    {
        cout << "Enter Age: ";
        getline(cin, ageStr);
        if(!isValidAge(ageStr))
            cout << "Invalid Age! Sirf numbers enter karein (1-149).\n";
    } while(!isValidAge(ageStr));
    age = stoi(ageStr);

    do
    {
        cout << "Enter CNIC (13 digits): ";
        getline(cin, cnic);

        if(!validCNIC(cnic))
        {
            cout << "Invalid CNIC!\n";
        }

    }while(!validCNIC(cnic));

    do
    {
        cout << "Enter Phone No (11 digits): ";
        getline(cin, phoneNo);

        if(!validPhone(phoneNo))
        {
            cout << "Invalid Phone Number!\n";
        }

    }while(!validPhone(phoneNo));

    cout << "Enter Gender (Male/Female): ";
    getline(cin, gender);
}

virtual void displayPerson() const
{
    cout << "\nName      : " << name;
    cout << "\nAge       : " << age;
    cout << "\nCNIC      : " << cnic;
    cout << "\nPhone No  : " << phoneNo;
    cout << "\nGender    : " << gender;
}

string getCNIC()   const { return cnic; }
string getGender() const { return gender; }
string getName()   const { return name; }

virtual ~Person(){}

};

//================ STAFF CLASS =================

class Staff : public Person
{
private:

static int staffCounter;

int    staffID;
string specialization;
string dutyDays;
string dutyTiming;

public:

Staff()
{
    staffID = ++staffCounter;
    specialization = "Cardiologist";
}

// Manual constructor for pre-loading default doctors
Staff(int id, string n, int a, string c,
      string ph, string g, string spec,
      string days, string timing)
{
    staffID        = id;
    name           = n;
    age            = a;
    cnic           = c;
    phoneNo        = ph;
    gender         = g;
    specialization = spec;
    dutyDays       = days;
    dutyTiming     = timing;
}

void addStaff()
{
	color(4);
    cout << "\n======= ADD STAFF =======\n";
    cout << "Staff ID       : " << staffID << endl;
    cout << "Specialization : Cardiologist (Fixed)\n";

    inputPerson();

    cout << "Enter Duty Day"
         << " (Monday/Tuesday/Wednesday"
         << "/Thursday/Friday/Saturday/Sunday): ";
    getline(cin, dutyDays);

    cout << "Enter Duty Timing (Morning/Evening): ";
    getline(cin, dutyTiming);

    cout << "\nStaff Added Successfully!\n";
}

void displayStaff() const
{
	color(5);
    cout << "\n================================";
    cout << "\nDoctor ID      : " << staffID;
    displayPerson();
    cout << "\nSpecialization : " << specialization;
    cout << "\nDuty Day       : " << dutyDays;
    cout << "\nDuty Timing    : " << dutyTiming;
    cout << "\n================================\n";
}

int    getID()             const { return staffID; }
string getSpecialization() const { return specialization; }
string getDutyDays()       const { return dutyDays; }
string getDutyTiming()     const { return dutyTiming; }

void saveToFile(ofstream &fout)
{
    fout << staffID        << endl;
    fout << name           << endl;
    fout << age            << endl;
    fout << cnic           << endl;
    fout << phoneNo        << endl;
    fout << gender         << endl;
    fout << specialization << endl;
    fout << dutyDays       << endl;
    fout << dutyTiming     << endl;
}

void loadFromFile(ifstream &fin)
{
    fin >> staffID;
    fin.ignore();
    getline(fin, name);
    fin >> age;
    fin.ignore();
    getline(fin, cnic);
    getline(fin, phoneNo);
    getline(fin, gender);
    getline(fin, specialization);
    getline(fin, dutyDays);
    getline(fin, dutyTiming);
}

};

int Staff::staffCounter = 5000;

//================ PATIENT CLASS =================

class Patient : public Person
{
protected:

static int patientCounter;
static int appointmentCounter;

int patientID;
int appointmentNo;

string date;
string day;
string timing;
string doctorGender;
string doctorName;
string billStatus;
string disease;

// ===== DATE TO DAY CONVERTER =====
string getDayFromDate(string dateStr)
{
    int year, month, dayNum;

    if(sscanf(dateStr.c_str(),
              "%d-%d-%d",
              &year, &month, &dayNum) != 3)
    {
        return "Invalid Date";
    }

    if(year < 1900 || month < 1 ||
       month > 12  || dayNum < 1 || dayNum > 31)
    {
        return "Invalid Date";
    }

    struct tm timeStruct = {};
    timeStruct.tm_year = year - 1900;
    timeStruct.tm_mon  = month - 1;
    timeStruct.tm_mday = dayNum;

    mktime(&timeStruct);

    string daysOfWeek[] = {
        "Sunday","Monday","Tuesday",
        "Wednesday","Thursday","Friday","Saturday"
    };

    return daysOfWeek[timeStruct.tm_wday];
}

public:

Patient()
{
    patientID     = ++patientCounter;
    appointmentNo = ++appointmentCounter;
}

virtual void addPatient()
{
	color(7);
    cout << "\n======= ADD PATIENT =======\n";
    cout << "Patient ID     : " << patientID << endl;
    cout << "Appointment No : " << appointmentNo << endl;

    inputPerson();

    bool validDate = false;
    while(!validDate)
    {
        cout << "Enter Appointment Date (YYYY-MM-DD): ";
        getline(cin, date);

        day = getDayFromDate(date);

        if(day == "Invalid Date")
        {
            cout << "Invalid date!"
                 << " Use format YYYY-MM-DD\n";
        }
        else
        {
            cout << "Day Detected : " << day << endl;
            validDate = true;
        }
    }

    cout << "Enter Timing (Morning/Evening): ";
    getline(cin, timing);

    cout << "Enter Doctor Gender Preference (Male/Female): ";
    getline(cin, doctorGender);

    cout << "Enter Bill Status (Paid/Unpaid): ";
    getline(cin, billStatus);

    cout << "Enter Disease: ";
    getline(cin, disease);
}

virtual void displayPatient() const
{
	color(8);
    cout << "\n================================";
    cout << "\nPatient ID      : " << patientID;
    cout << "\nAppointment No  : " << appointmentNo;
    displayPerson();
    cout << "\nDate            : " << date;
    cout << "\nDay             : " << day;
    cout << "\nTiming          : " << timing;
    cout << "\nDoctor Gender   : " << doctorGender;
    cout << "\nDoctor Assigned : " << doctorName;
    cout << "\nBill Status     : " << billStatus;
    cout << "\nDisease         : " << disease;
    cout << "\n================================\n";
}

// GETTERS
string getPatientCNIC()  const { return cnic; }
string getDay()          const { return day; }
string getTiming()       const { return timing; }
string getDoctorGender() const { return doctorGender; }

// SETTERS
void setDoctorName(string d) { doctorName = d; }
void setName(string n)       { name = n; }
void setAge(int a)           { age = a; }
void setCNIC(string c)       { cnic = c; }
void setPhone(string p)      { phoneNo = p; }
void setGender(string g)     { gender = g; }
void setDate(string d)       { date = d; }
void setDay(string d)        { day = d; }
void setTiming(string t)     { timing = t; }
void setBillStatus(string b) { billStatus = b; }
void setDisease(string d)    { disease = d; }

// HISTORY
void saveHistory()
{
    ofstream fout("history.txt", ios::app);

    fout << "Patient ID     : " << patientID     << endl;
    fout << "Appointment No : " << appointmentNo << endl;
    fout << "Name           : " << name          << endl;
    fout << "CNIC           : " << cnic          << endl;
    fout << "Date           : " << date          << endl;
    fout << "Day            : " << day           << endl;
    fout << "Doctor         : " << doctorName    << endl;
    fout << "Bill Status    : " << billStatus    << endl;
    fout << "Disease        : " << disease       << endl;
    fout << "------------------------" << endl;

    fout.close();
}

// FILE SAVE
void saveToFile(ofstream &fout)
{
    fout << patientID     << endl;
    fout << appointmentNo << endl;
    fout << name          << endl;
    fout << age           << endl;
    fout << cnic          << endl;
    fout << phoneNo       << endl;
    fout << gender        << endl;
    fout << date          << endl;
    fout << day           << endl;
    fout << timing        << endl;
    fout << doctorGender  << endl;
    fout << doctorName    << endl;
    fout << billStatus    << endl;
    fout << disease       << endl;
}

// FILE LOAD
void loadFromFile(ifstream &fin)
{
    fin >> patientID;
    fin >> appointmentNo;
    fin.ignore();
    getline(fin, name);
    fin >> age;
    fin.ignore();
    getline(fin, cnic);
    getline(fin, phoneNo);
    getline(fin, gender);
    getline(fin, date);
    getline(fin, day);
    getline(fin, timing);
    getline(fin, doctorGender);
    getline(fin, doctorName);
    getline(fin, billStatus);
    getline(fin, disease);
}

};

int Patient::patientCounter     = 1000;
int Patient::appointmentCounter = 00;

//================ ADMITTED PATIENT =================

class AdmittedPatient : public Patient
{
private:

string admissionDate;
string dischargeDate;

public:

void admitPatient()
{
	color(1);
    cout << "\n======= ADMIT PATIENT =======\n";

    addPatient();

    cout << "Enter Admission Date (YYYY-MM-DD): ";
    getline(cin, admissionDate);

    cout << "Enter Discharge Date (YYYY-MM-DD): ";
    getline(cin, dischargeDate);
}

void displayAdmittedPatient() const
{
    displayPatient();
    cout << "Admission Date : " << admissionDate << endl;
    cout << "Discharge Date : " << dischargeDate << endl;
}

void setAdmissionDate(string d) { admissionDate = d; }
void setDischargeDate(string d) { dischargeDate = d; }

void saveAdmitted(ofstream &fout)
{
    saveToFile(fout);
    fout << admissionDate << endl;
    fout << dischargeDate << endl;
}

void loadAdmitted(ifstream &fin)
{
    loadFromFile(fin);
    getline(fin, admissionDate);
    getline(fin, dischargeDate);
}

};

//================ HOSPITAL CLASS =================

class Hospital
{
private:

vector<Staff>           doctors;
vector<Patient>         patients;
vector<AdmittedPatient> admittedPatients;

// ===== LOWERCASE HELPER =====
string toLower(string str)
{
    for(char &c : str)
        c = tolower(c);
    return str;
}

// ===== DOCTOR ASSIGNMENT =====
string assignDoctor(string patientDay,
                    string patientTiming,
                    string patientDoctorGender)
{
    string doctorAssigned = "Doctor Not Available";

    for(auto &d : doctors)
    {
        bool genderMatch =
            (toLower(d.getGender())
             == toLower(patientDoctorGender));

        bool dayMatch =
            (toLower(d.getDutyDays())
             == toLower(patientDay));

        bool timingMatch =
            (toLower(d.getDutyTiming())
             == toLower(patientTiming));

        if(genderMatch && dayMatch && timingMatch)
        {
            doctorAssigned = d.getName();
            break;
        }
    }

    return doctorAssigned;
}

// ===== LOAD DEFAULT DOCTORS =====
void loadDefaultDoctors()
{
    // ----- MALE DOCTORS -----
    doctors.push_back(Staff(5001, "Dr. Ahmed Ali",      45, "1234567890123", "03001234567", "Male",   "Cardiologist", "Monday",    "Morning"));
    doctors.push_back(Staff(5002, "Dr. Usman Khan",     50, "1234567890124", "03001234568", "Male",   "Cardiologist", "Monday",    "Evening"));
    doctors.push_back(Staff(5003, "Dr. Bilal Hassan",   40, "1234567890125", "03001234569", "Male",   "Cardiologist", "Tuesday",   "Morning"));
    doctors.push_back(Staff(5004, "Dr. Kamran Malik",   38, "1234567890126", "03001234570", "Male",   "Cardiologist", "Tuesday",   "Evening"));
    doctors.push_back(Staff(5005, "Dr. Tariq Mehmood",  55, "1234567890127", "03001234571", "Male",   "Cardiologist", "Wednesday", "Morning"));
    doctors.push_back(Staff(5006, "Dr. Zafar Iqbal",    48, "1234567890128", "03001234572", "Male",   "Cardiologist", "Wednesday", "Evening"));
    doctors.push_back(Staff(5007, "Dr. Imran Siddiqui", 42, "1234567890129", "03001234573", "Male",   "Cardiologist", "Thursday",  "Morning"));
    doctors.push_back(Staff(5008, "Dr. Naveed Akhtar",  36, "1234567890130", "03001234574", "Male",   "Cardiologist", "Thursday",  "Evening"));
    doctors.push_back(Staff(5009, "Dr. Sajid Rehman",   44, "1234567890131", "03001234575", "Male",   "Cardiologist", "Friday",    "Morning"));
    doctors.push_back(Staff(5010, "Dr. Faisal Raza",    39, "1234567890132", "03001234576", "Male",   "Cardiologist", "Friday",    "Evening"));
    doctors.push_back(Staff(5011, "Dr. Hamid Javed",    52, "1234567890133", "03001234577", "Male",   "Cardiologist", "Saturday",  "Morning"));
    doctors.push_back(Staff(5012, "Dr. Waseem Butt",    46, "1234567890134", "03001234578", "Male",   "Cardiologist", "Saturday",  "Evening"));
    doctors.push_back(Staff(5013, "Dr. Adeel Chaudhry", 41, "1234567890135", "03001234579", "Male",   "Cardiologist", "Sunday",    "Morning"));
    doctors.push_back(Staff(5014, "Dr. Rizwan Shah",    37, "1234567890136", "03001234580", "Male",   "Cardiologist", "Sunday",    "Evening"));

    // ----- FEMALE DOCTORS -----
    doctors.push_back(Staff(5015, "Dr. Ayesha Noor",    43, "1234567890137", "03011234567", "Female", "Cardiologist", "Monday",    "Morning"));
    doctors.push_back(Staff(5016, "Dr. Sana Fatima",    38, "1234567890138", "03011234568", "Female", "Cardiologist", "Monday",    "Evening"));
    doctors.push_back(Staff(5017, "Dr. Hina Malik",     45, "1234567890139", "03011234569", "Female", "Cardiologist", "Tuesday",   "Morning"));
    doctors.push_back(Staff(5018, "Dr. Rabia Tariq",    40, "1234567890140", "03011234570", "Female", "Cardiologist", "Tuesday",   "Evening"));
    doctors.push_back(Staff(5019, "Dr. Zara Ahmed",     35, "1234567890141", "03011234571", "Female", "Cardiologist", "Wednesday", "Morning"));
    doctors.push_back(Staff(5020, "Dr. Nadia Khan",     50, "1234567890142", "03011234572", "Female", "Cardiologist", "Wednesday", "Evening"));
    doctors.push_back(Staff(5021, "Dr. Madiha Iqbal",   42, "1234567890143", "03011234573", "Female", "Cardiologist", "Thursday",  "Morning"));
    doctors.push_back(Staff(5022, "Dr. Amna Butt",      36, "1234567890144", "03011234574", "Female", "Cardiologist", "Thursday",  "Evening"));
    doctors.push_back(Staff(5023, "Dr. Farah Siddiqui", 48, "1234567890145", "03011234575", "Female", "Cardiologist", "Friday",    "Morning"));
    doctors.push_back(Staff(5024, "Dr. Kiran Javed",    39, "1234567890146", "03011234576", "Female", "Cardiologist", "Friday",    "Evening"));
    doctors.push_back(Staff(5025, "Dr. Uzma Rehman",    44, "1234567890147", "03011234577", "Female", "Cardiologist", "Saturday",  "Morning"));
    doctors.push_back(Staff(5026, "Dr. Sadia Raza",     41, "1234567890148", "03011234578", "Female", "Cardiologist", "Saturday",  "Evening"));
    doctors.push_back(Staff(5027, "Dr. Lubna Hassan",   37, "1234567890149", "03011234579", "Female", "Cardiologist", "Sunday",    "Morning"));
    doctors.push_back(Staff(5028, "Dr. Munira Shah",    46, "1234567890150", "03011234580", "Female", "Cardiologist", "Sunday",    "Evening"));
}

public:

Hospital()
{
    loadDefaultDoctors();
}

// ===== DUPLICATE CNIC CHECK =====
bool CNICExists(string cnic)
{
    for(auto &p : patients)
        if(p.getPatientCNIC() == cnic)
            return true;

    for(auto &a : admittedPatients)
        if(a.getPatientCNIC() == cnic)
            return true;

    return false;
}

// ===== ADD STAFF =====
void addStaff()
{
    Staff s;
    s.addStaff();
    doctors.push_back(s);
    cout << "\nDoctor Added Successfully!\n";
    s.displayStaff();
    system("pause");
    system("cls");
}

// ===== VIEW STAFF =====
void viewStaff()
{
	color(4);
    cout << "\n======= ALL DOCTORS =======\n";

    if(doctors.empty())
    {
        cout << "\nNo Doctor Record Found!\n";
        system("pause");
        system("cls");
        return;
    }

    for(auto &d : doctors)
        d.displayStaff();

    system("pause");
    system("cls");
}

// ===== ADD PATIENT =====
void addPatient()
{
    Patient p;
    p.addPatient();

    if(CNICExists(p.getPatientCNIC()))
    {
        cout << "\nCNIC Already Exists!\n";
        system("pause");
        system("cls");
        return;
    }

    string doctorAssigned =
        assignDoctor(
            p.getDay(),
            p.getTiming(),
            p.getDoctorGender()
        );

    p.setDoctorName(doctorAssigned);
    p.saveHistory();
    patients.push_back(p);
    color(6);
    cout << "\nAppointment Created Successfully!\n";
    cout << "\nDoctor Allotted : " << doctorAssigned << endl;

    p.displayPatient();
    system("pause");
    system("cls");
}

// ===== ADMIT PATIENT =====
void admitPatient()
{
    AdmittedPatient ap;
    ap.admitPatient();

    if(CNICExists(ap.getPatientCNIC()))
    {
        cout << "\nCNIC Already Exists!\n";
        system("pause");
        system("cls");
        return;
    }

    string doctorAssigned =
        assignDoctor(
            ap.getDay(),
            ap.getTiming(),
            ap.getDoctorGender()
        );

    ap.setDoctorName(doctorAssigned);
    admittedPatients.push_back(ap);
    color(6);
    cout << "\nPatient Admitted Successfully!\n";
    color(9);
    cout << "\nDoctor Allotted : " << doctorAssigned << endl;

    ap.displayAdmittedPatient();
    system("pause");
    system("cls");
}

// ===== SAVE DATA =====
void saveData()
{
    ofstream fout2("patients.txt");
    fout2 << patients.size() << endl;
    for(auto &p : patients)
        p.saveToFile(fout2);
    fout2.close();

    ofstream fout3("admitted.txt");
    fout3 << admittedPatients.size() << endl;
    for(auto &a : admittedPatients)
        a.saveAdmitted(fout3);
    fout3.close();
    color(10);
    cout << "\nData Saved Successfully!\n";
}

// ===== LOAD DATA =====
void loadData()
{
    ifstream fin2("patients.txt");
    if(fin2)
    {
        int total;
        fin2 >> total;
        fin2.ignore();
        for(int i = 0; i < total; i++)
        {
            Patient p;
            p.loadFromFile(fin2);
            patients.push_back(p);
        }
        fin2.close();
    }

    ifstream fin3("admitted.txt");
    if(fin3)
    {
      