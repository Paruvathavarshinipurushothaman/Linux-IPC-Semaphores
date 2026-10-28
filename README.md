# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.
```

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/wait.h>

#define NUM_LOOPS 10

union semun {
    int val;
    struct semid_ds *buf;
    unsigned short int *array;
    struct seminfo *__buf;
};

void wait_semaphore(int sem_id)
{
    struct sembuf op = {0, -1, 0};
    if (semop(sem_id, &op, 1) == -1) {
        perror("semop wait");
        exit(1);
    }
}

void signal_semaphore(int sem_id)
{
    struct sembuf op = {0, 1, 0};
    if (semop(sem_id, &op, 1) == -1) {
        perror("semop signal");
        exit(1);
    }
}

int main(void)
{
    int sem_id;
    pid_t child_pid;
    union semun sem_val;

    sem_id = semget(IPC_PRIVATE, 1, 0600);
    if (sem_id == -1) {
        perror("semget");
        return 1;
    }

    printf("Semaphore set created, ID: %d\n", sem_id);

    sem_val.val = 0;
    if (semctl(sem_id, 0, SETVAL, sem_val) == -1) {
        perror("semctl");
        semctl(sem_id, 0, IPC_RMID);
        return 1;
    }

    child_pid = fork();

    if (child_pid < 0) {
        perror("fork");
        semctl(sem_id, 0, IPC_RMID);
        return 1;
    }

    if (child_pid == 0) {
        /* Consumer process */
        for (int i = 0; i < NUM_LOOPS; i++) {
            wait_semaphore(sem_id);
            printf("Consumer: %d\n", i);
            fflush(stdout);
        }
        _exit(0);
    } else {
        /* Producer process */
        for (int i = 0; i < NUM_LOOPS; i++) {
            printf("Producer: %d\n", i);
            fflush(stdout);
            signal_semaphore(sem_id);
            usleep(500000);
        }

        wait(NULL);
        semctl(sem_id, 0, IPC_RMID);
        printf("Semaphore removed.\n");
    }

    return 0;
}
```

## OUTPUT
## $ ./sem.o 
<img width="1073" height="627" alt="image" src="https://github.com/user-attachments/assets/982f98c0-f205-4da1-bdf3-5e8e766e02c3" />


## $ ipcs
<img width="988" height="247" alt="image" src="https://github.com/user-attachments/assets/e8e1c218-d995-4ab4-a702-d97addf5d3f8" />

# RESULT:
The program is executed successfully.
