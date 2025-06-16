#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

#define PAGE_SIZE 64
#define TOTAL_MEMORY 1024
#define MAX_FRAMES (TOTAL_MEMORY / PAGE_SIZE)
#define MAX_PROCESSES 20

typedef struct {
    int id;
    int arrival_time;
    int size;
    int allocated; // 1 if allocated, 0 if not
} Process;

int memory[MAX_FRAMES];           // Stores process IDs in memory
int frame_queue[MAX_FRAMES];      // FIFO frame order
int queue_front = 0, queue_rear = 0;

int current_time = 0;

// Load processes from file and check minimum count
int load_processes(const char *filename, Process processes[]) {
    FILE *file = fopen(filename, "r");
    if (!file) {
        printf("Error: Unable to open file.\n");
        exit(1);
    }

    int count = 0;
    while (fscanf(file, "%d %d %d", &processes[count].id,
                  &processes[count].arrival_time,
                  &processes[count].size) == 3) {
        if (count >= MAX_PROCESSES) {
            printf("Error: Cannot load more than %d processes.\n", MAX_PROCESSES);
            fclose(file);
            exit(1);
        }
        processes[count].allocated = 0;
        count++;
    }

    fclose(file);

    if (count < 10) {
        printf("Error: At least 10 processes required in processes.txt\n");
        exit(1);
    }

    return count;
}

// Display memory state
void display_memory() {
    printf("\nMemory State [Time: %d]:\n", current_time);
    for (int i = 0; i < MAX_FRAMES; i++) {
        if (memory[i] == -1)
            printf("[Free] ");
        else
            printf("[P%d] ", memory[i]);
    }
    printf("\n");
}

// Enqueue a frame to FIFO
void enqueue_frame(int frame_index) {
    frame_queue[queue_rear % MAX_FRAMES] = frame_index;
    queue_rear++;
}

// Dequeue the oldest frame from FIFO
int dequeue_frame() {
    if (queue_front == queue_rear)
        return -1;
    int frame_index = frame_queue[queue_front % MAX_FRAMES];
    queue_front++;
    return frame_index;
}

// Allocate memory using FIFO
int allocate_process(Process process) {
    int pages_needed = (process.size + PAGE_SIZE - 1) / PAGE_SIZE;
    int pages_allocated = 0;

    // Try to allocate from free frames
    for (int i = 0; i < MAX_FRAMES && pages_allocated < pages_needed; i++) {
        if (memory[i] == -1) {
            memory[i] = process.id;
            enqueue_frame(i);
            pages_allocated++;
        }
    }

    // Use FIFO replacement if needed
    while (pages_allocated < pages_needed) {
        int frame_to_replace = dequeue_frame();
        if (frame_to_replace == -1) break;

        printf("Evicting P%d from frame %d (FIFO)\n", memory[frame_to_replace], frame_to_replace);
        memory[frame_to_replace] = process.id;
        enqueue_frame(frame_to_replace);
        pages_allocated++;
    }

    return (pages_allocated == pages_needed);
}

// Calculate average, min, max process sizes
void calculate_stats(Process processes[], int num) {
    int total = 0, max = 0, min = INT_MAX;
    for (int i = 0; i < num; i++) {
        total += processes[i].size;
        if (processes[i].size > max) max = processes[i].size;
        if (processes[i].size < min) min = processes[i].size;
    }
    printf("\nProcess Size Stats: Avg: %.2f MB, Min: %d MB, Max: %d MB\n\n",
           (float)total / num, min, max);
}

// Main simulation loop
void simulate(Process processes[], int num_processes) {
    int allocated_count = 0;

    while (allocated_count < num_processes) {
        printf("\nTime: %d\n", current_time);

        for (int i = 0; i < num_processes; i++) {
            if (!processes[i].allocated && processes[i].arrival_time <= current_time) {
                int success = allocate_process(processes[i]);
                if (success) {
                    printf("Allocated memory for process P%d\n", processes[i].id);
                    processes[i].allocated = 1;
                    allocated_count++;
                } else {
                    printf("Failed to allocate memory for process P%d\n", processes[i].id);
                    processes[i].allocated = 1; // mark as done to prevent retry
                }
            }
        }

        display_memory();
        current_time++;
    }
}

int main() {
    Process processes[MAX_PROCESSES];

    // Initialize memory and frame queue
    for (int i = 0; i < MAX_FRAMES; i++) {
        memory[i] = -1;
    }

    int num_processes = load_processes("processes.txt", processes);
    calculate_stats(processes, num_processes);
    simulate(processes, num_processes);

    return 0;
}
