#include <mpi.h>
#include <iostream>
#include <algorithm>
#include <cstdlib>
#include <ctime>

using namespace std;

int main(int argc, char* argv[]) {
    int rank, size, n = 100000;
    int* arr = nullptr;
    int* local_arr;
    int local_n;

    MPI_Init(&argc, &argv); // Initialize MPI
    MPI_Comm_rank(MPI_COMM_WORLD, &rank); // Get current process rank
    MPI_Comm_size(MPI_COMM_WORLD, &size); // Get total number of processes

    local_n = n / size;

    if (rank == 0) {
        arr = new int[n];
        srand(time(NULL));
        for (int i = 0; i < n; i++)
            arr[i] = rand() % 10000;
    }

    local_arr = new int[local_n];

    double start_time = MPI_Wtime();

    // Scatter array parts to all processes
    MPI_Scatter(arr, local_n, MPI_INT, local_arr, local_n, MPI_INT, 0, MPI_COMM_WORLD);

    // Each process sorts its part
    std::sort(local_arr, local_arr + local_n);

    // Gather sorted parts back to root
    MPI_Gather(local_arr, local_n, MPI_INT, arr, local_n, MPI_INT, 0, MPI_COMM_WORLD);

    double end_time = MPI_Wtime();

    if (rank == 0) {
        // Merge sorted parts sequentially
        int* merged = new int[n];
        std::merge(arr, arr + local_n, arr + local_n, arr + 2 * local_n, merged);
        for (int i = 2; i < size; i++) {
            std::merge(merged, merged + i * local_n, arr + i * local_n, arr + (i + 1) * local_n, merged);
        }

        cout << "MPI QuickSort completed in " << (end_time - start_time) << " seconds.\n";

        // Optional: Check sorted correctness
        /*
        bool sorted = true;
        for (int i = 0; i < n - 1; i++) {
            if (merged[i] > merged[i + 1]) {
                sorted = false;
                break;
            }
        }
        cout << "Array sorted: " << (sorted ? "Yes" : "No") << endl;
        */

        delete[] merged;
        delete[] arr;
    }

    delete[] local_arr;

    MPI_Finalize(); // Finalize MPI
    return 0;
}
