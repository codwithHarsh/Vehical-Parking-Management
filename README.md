# Vehical-Parking-Management

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MAX_PARK 5
#define MAX_WAIT 5


struct Vehicle {
    char number[20];
    time_t inTime;
};


struct Vehicle parking[MAX_PARK];
int top = -1;


char waiting[MAX_WAIT][20];
int front = 0, rear = -1, waitCount = 0;


struct Log {
    char number[20];
    time_t inTime;
    time_t outTime;
    struct Log* next;
};

struct Log* logHead = NULL;


int isParkingFull() {
    return top == MAX_PARK - 1;
}

int isParkingEmpty() {
    return top == -1;
}

void parkVehicle(char num[]) {
    top++;
    strcpy(parking[top].number, num);
    parking[top].inTime = time(NULL);
}


int isWaitFull() {
    return waitCount == MAX_WAIT;
}

int isWaitEmpty() {
    return waitCount == 0;
}

void addToWaiting(char num[]) {
    if (isWaitFull()) return;
    rear = (rear + 1) % MAX_WAIT;
    strcpy(waiting[rear], num);
    waitCount++;
}

void removeFromWaiting(char* num) {
    if (isWaitEmpty()) return;
    strcpy(num, waiting[front]);
    front = (front + 1) % MAX_WAIT;
    waitCount--;
}

void addLog(char num[], time_t inTime, time_t outTime) {
    struct Log* newLog = (struct Log*)malloc(sizeof(struct Log));
    strcpy(newLog->number, num);
    newLog->inTime = inTime;
    newLog->outTime = outTime;
    newLog->next = logHead;
    logHead = newLog;
}

int calculateCharge(time_t in, time_t out) {
    int minutes = (int)(difftime(out, in) / 60);
    if (minutes == 0) minutes = 1;
    return minutes * 10;  
}

void showAvailableSlots() {
    printf("Available parking slots: %d\n", MAX_PARK - top - 1);
}

void showWaitingQueue() {
    if (isWaitEmpty()) {
        printf("Waiting queue is empty.\n");
        return;
    }
    printf("Waiting queue:\n");
    int i, index = front;
    for (i = 0; i < waitCount; i++) {
        printf("- %s\n", waiting[index]);
        index = (index + 1) % MAX_WAIT;
    }
}

void showLogs() {
    struct Log* temp = logHead;
    if (temp == NULL) {
        printf("No vehicle logs yet.\n");
        return;
    }
    printf("Vehicle Logs:\n");
    while (temp != NULL) {
        printf("Vehicle: %s\n", temp->number);
        printf("In Time: %s", ctime(&temp->inTime));
        printf("Out Time: %s", ctime(&temp->outTime));
        printf("Charge: ₹%d\n", calculateCharge(temp->inTime, temp->outTime));
        printf("-----------------------------\n");
        temp = temp->next;
    }
}

int main() {
    int choice;
    char number[20];

    while (1) {
        printf("\n---- Vehicle Parking System ----\n");
        printf("1. Park Vehicle\n");
        printf("2. Unpark Vehicle\n");
        printf("3. Show Available Slots\n");
        printf("4. Show Waiting Queue\n");
        printf("5. Show Vehicle Logs\n");
        printf("6. Exit\n");
        printf("Choose option: ");
        scanf("%d", &choice);

        if (choice == 1) {
            printf("Enter Vehicle Number: ");
            scanf("%s", number);

            if (!isParkingFull()) {
                parkVehicle(number);
                printf("Vehicle parked.\n");
            } else if (!isWaitFull()) {
                addToWaiting(number);
                printf("Parking full. Added to waiting queue.\n");
            } else {
                printf("Parking and waiting full.\n");
            }
        }

        else if (choice == 2) {
            if (!isParkingEmpty()) {
                struct Vehicle leaving = parking[top];
                top--;

                time_t outTime = time(NULL);
                int charge = calculateCharge(leaving.inTime, outTime);
                printf("Vehicle %s unparked. Charge: ₹%d\n", leaving.number, charge);
                addLog(leaving.number, leaving.inTime, outTime);

                if (!isWaitEmpty()) {
                    char nextNum[20];
                    removeFromWaiting(nextNum);
                    parkVehicle(nextNum);
                    printf("Vehicle %s from waiting parked.\n", nextNum);
                }
            } else {
                printf("No vehicles to unpark.\n");
            }
        }

        else if (choice == 3) {
            showAvailableSlots();
        }

        else if (choice == 4) {
            showWaitingQueue();
        }

        else if (choice == 5) {
            showLogs();
        }

        else if (choice == 6) {
            printf("Exiting program.\n");
            break;
        }

        else {
            printf("Invalid choice. Try again.\n");
        }
    }

    return 0;
}
