# Railway-Reservation-System by the concept of data structure using c++

#include <iostream>
#include <queue>
#include <string>
#include <vector>
#include <map>
#include <iomanip>
#include <algorithm>
using namespace std;

struct Passenger {
    int reg_no;
    int age;
    string name;
    string seat; 
    Passenger* next;
};

struct Train {
    int train_no;
    string name;
    string source;
    string destination;
    float distance;
    vector<string> seats;  
    map<string, Passenger*> reservedSeats; 
};

class RailwayReservation {
private:
    map<int, Train> trains;      
    Passenger* start;            
    queue<Passenger*> waitingList; 
    int num;                     
    string role;                 

public:
    RailwayReservation() : start(nullptr), num(0), role("") {}

    void authenticate() {
        int choice;
        cout << "Login as:\n1. Admin\n2. Passenger\nEnter choice: ";
        cin >> choice;
        if (choice == 1) role = "Admin";
        else if (choice == 2) role = "Passenger";
        else {
            cout << "Invalid choice! Exiting...\n";
            exit(0);
        }
    }

    bool isAdmin() const {
        return role == "Admin";
    }

    void addTrain() {
        if (!isAdmin()) {
            cout << "Unauthorized access!\n";
            return;
        }
        Train train;
        cout << "Enter Train Number: ";
        cin >> train.train_no;
        cin.ignore();
        cout << "Enter Train Name: ";
        getline(cin, train.name);
        cout << "Enter Source: ";
        cin >> train.source;
        cout << "Enter Destination: ";
        cin >> train.destination;
        cout << "Enter Distance (in km): ";
        cin >> train.distance;
        int totalSeats;
        cout << "Enter Total Number of Seats: ";
        cin >> totalSeats;
        for (int i = 1; i <= totalSeats; i++) {
            train.seats.push_back("S" + to_string(i));
        }
        trains[train.train_no] = train;
        cout << "Train added successfully with " << totalSeats << " seats!\n";
    }

    int bookTicket() {
        if (role != "Passenger") {
            cout << "Unauthorized access!\n";
            return -1;
        }
        if (trains.empty()) {
            cout << "No trains available for booking!\n";
            return -1;
        }

        cout << "Available Trains:\n";
        for (const auto& t : trains) {
            const Train& train = t.second;
            cout << "Train No: " << train.train_no
                 << ", Name: " << train.name
                 << ", Source: " << train.source
                 << ", Destination: " << train.destination
                 << ", Distance: " << train.distance << " km\n";
        }

        int trainNo;
        cout << "Enter Train Number to book: ";
        cin >> trainNo;

        if (trains.find(trainNo) == trains.end()) {
            cout << "Invalid Train Number!\n";
            return -1;
        }

        Train& train = trains[trainNo];
        if (train.seats.empty()) {
            cout << "No seats available in this train. Adding to waiting list...\n";
            Passenger* newPassenger = createPassenger();
            if (newPassenger) {
                waitingList.push(newPassenger);
            }
            return 0;
        }

        cout << "Available Seats: ";
        for (const string& seat : train.seats) {
            cout << seat << " ";
        }
        cout << "\nEnter Seat Number to book: ";
        string seatChoice;
        cin >> seatChoice;

        auto it = find(train.seats.begin(), train.seats.end(), seatChoice);
        if (it == train.seats.end()) {
            cout << "Invalid or already reserved seat!\n";
            return -1;
        }

        Passenger* newPassenger = createPassenger();
        if (!newPassenger) return -1;

        newPassenger->seat = seatChoice;
        train.seats.erase(it);
        train.reservedSeats[seatChoice] = newPassenger;

        if (!start) {
            start = newPassenger;
        } else {
            Passenger* temp = start;
            while (temp->next) temp = temp->next;
            temp->next = newPassenger;
        }
        num++;
        cout << "Booking Successful! Seat: " << newPassenger->seat << "\n";
        return 1;
    }

    void cancelBooking() {
        if (role != "Passenger") {
            cout << "Unauthorized access!\n";
            return;
        }

        int trainNo;
        string seatNo;
        cout << "Enter Train Number: ";
        cin >> trainNo;
        cout << "Enter Seat Number to cancel: ";
        cin >> seatNo;

        if (trains.find(trainNo) == trains.end()) {
            cout << "Invalid Train Number!\n";
            return;
        }

        Train& train = trains[trainNo];
        auto it = train.reservedSeats.find(seatNo);
        if (it == train.reservedSeats.end()) {
            cout << "No booking found for the provided seat!\n";
            return;
        }

        Passenger* passenger = it->second;
        train.seats.push_back(seatNo); 
        train.reservedSeats.erase(it); 

        Passenger* temp = start;
        Passenger* prev = nullptr;
        while (temp && temp != passenger) {
            prev = temp;
            temp = temp->next;
        }

        if (temp) {
            if (prev) {
                prev->next = temp->next;
            } else {
                start = temp->next;
            }
            delete temp;
            num--;
            cout << "Booking cancelled successfully!\n";
        }
    }

    void displayTrains() const {
        if (!isAdmin()) {
            cout << "Unauthorized access!\n";
            return;
        }
        cout << "\nTrain Schedules:\n";
        for (const auto& t : trains) {
            const Train& train = t.second;
            cout << "Train No: " << train.train_no
                 << ", Name: " << train.name
                 << ", Source: " << train.source
                 << ", Destination: " << train.destination
                 << ", Distance: " << train.distance << " km, Seats Available: " << train.seats.size() << "\n";
        }
    }

private:
    Passenger* createPassenger() {
        Passenger* newPassenger = new Passenger;
        cout << "Enter Name: ";
        cin >> newPassenger->name;
        cout << "Enter Age: ";
        cin >> newPassenger->age;
        if (newPassenger->age >= 90 || newPassenger->age <= 10) {
            cout << "Age not eligible for booking!\n";
            delete newPassenger;
            return nullptr;
        }
        newPassenger->reg_no = ++num;
        newPassenger->next = nullptr;
        return newPassenger;
    }
};

int main() {
    RailwayReservation rr;

    while (true) {
        rr.authenticate();
        while (true) {
            int choice;
            cout << "\nMenu:\n";
            if (rr.isAdmin()) {
                cout << "1. Add Train\n2. Display Train Schedules\n3. Logout\n";
                cin >> choice;

                if (choice == 1) {
                    rr.addTrain();
                } else if (choice == 2) {
                    rr.displayTrains();
                } else if (choice == 3) {
                    cout << "Logging out...\n";
                    break;
                } else {
                    cout << "Invalid choice!\n";
                }
            } else {
                cout << "1. Book Ticket\n2. Cancel Booking\n3. Logout\n";
                cin >> choice;

                if (choice == 1) {
                    rr.bookTicket();
                } else if (choice == 2) {
                    rr.cancelBooking();
                } else if (choice == 3) {
                    cout << "Logging out...\n";
                    break;
                } else {
                    cout << "Invalid choice!\n";
                }
            }
        }
    }

    return 0;
}
