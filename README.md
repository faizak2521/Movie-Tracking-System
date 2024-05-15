# Movie-Tracking-System
A movie tracking system that handles information about movies, using a binary file.
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>


#define MAX_TITLE_LENGTH 21 // Maximum title length
#define NUM_RECORDS 10      // Defines number of records

// Define enumeration for movie genres
typedef enum {
    ACTION, ANIMATION, DRAMA, SCI_FI, SUPERHERO
} Genre;

// Define structure to store movie information
typedef struct {
    char title[MAX_TITLE_LENGTH]; // Array to store the title of the movie
    int downloads;                // Integer to store the number of downloads
    Genre genre;                  // Variable of Genre type to store the movie genre
    int isEmpty;                  // Integer to check if the record is empty
} Movie;


// Declaration of functions used in the program
FILE* openFile();
void addMovie(FILE *fptr);
void deleteMovie(FILE *fptr);
void updateMovie(FILE *fptr);
void displayMovieInfo(FILE *fptr);
int displayMenu();


// Main function
int main() {
    FILE *fptr = openFile(); // Call openFile function to open the file and store the returned pointer in fptr
    int choice; // Variable to store user's menu choice

    // Loop to show menu and process choices until user exits
    while(1) {
        choice = displayMenu(); // Display menu and get user choice
        switch(choice) { // Switch statement to process choice
            case 1:
                addMovie(fptr); // Call addMovie function if choice is 1
                break;
            case 2:
                deleteMovie(fptr); // Call deleteMovie function if choice is 2
                break;
            case 3:
                updateMovie(fptr); // Call updateMovie function if choice is 3
                break;
            case 4:
                displayMovieInfo(fptr); // Call displayMovieInfo function if choice is 4
                break;
            case 5:
                fclose(fptr); // Close the file pointer if choice is 5
                printf("Exiting program.\n"); // Print exit message
                return 0; // Return 0 to exit the program
            default:
                printf("Invalid choice. Please try again.\n"); // Print error message if choice is invalid
        }
    }
}


// Function 1 to open the movie file and return the file pointer
FILE* openFile() {
    FILE *fptr = fopen("movies.dat", "wb+"); // Open or create the file for reading and writing
    if (!fptr) { // Check if file opening failed
        perror("Unable to open file"); // Print error message
        exit(EXIT_FAILURE); // Exit the program with an error state
    }

    Movie blankMovie = {"", 0, 0, 1}; // Create a blank movie record
    for (int i = 0; i < NUM_RECORDS; i++) { // Loop to initialize the file with blank records
        fwrite(&blankMovie, sizeof(Movie), 1, fptr); // Write blank movie to file
    }
    fclose(fptr); // Close and reopen file to reset the pointer

    fptr = fopen("movies.dat", "rb+"); // Reopen the file for reading and writing
    if (!fptr) { // Check if file opening failed
        perror("Unable to open file"); // Print error message
        exit(EXIT_FAILURE); // Exit the program with an error state
    }
    return fptr; // Return the file pointer
}


// Function 2 to add a movie to the file
void addMovie(FILE *fptr) {
    Movie m; // Declare a movie structure to store new movie data
    printf("Enter title of movie (max 20 chars): "); // Prompt user for movie title
    scanf(" %[^\n]%*c", m.title); // Read the title including spaces

    // Check if the title length is too long
    if (strlen(m.title) >= MAX_TITLE_LENGTH) {
        m.title[MAX_TITLE_LENGTH - 1] = '\0'; // Truncate the title if too long
        printf("Title exceeds 20 characters and will be truncated\n"); // Inform user
    }

    printf("Enter number of downloads: "); // Prompt user for number of downloads
    scanf("%d%*c", &m.downloads); // Read the number of downloads

    printf("Enter genre (0: Action, 1: Animation, 2: Drama, 3: Sci-Fi, 4: Superhero): "); // Prompt user for genre
    int genre;
    scanf("%d%*c", &genre); // Read the genre
    while (genre < 0 || genre > 4) { // Validate genre input
        printf("Invalid genre. Enter a valid genre number (0-4): "); // Prompt again if invalid
        scanf("%d%*c", &genre); // Read genre again
    }
    m.genre = genre; // Set the genre of the movie
    m.isEmpty = 0; // Mark the record as not empty

    // Find the first empty record to add the new movie
    fseek(fptr, 0, SEEK_SET); // Move to the start of the file
    Movie temp; // Temporary movie structure to read movies from file
    int added = 0; // Flag to check if movie was added

    // Loop through records to find an empty spot
    for (int i = 0; i < NUM_RECORDS && !added; i++) {
        fread(&temp, sizeof(Movie), 1, fptr); // Read a movie from the file
        if (temp.isEmpty) { // Check if the movie record is empty
            fseek(fptr, -sizeof(Movie), SEEK_CUR); // Move back to the position of the empty record
            fwrite(&m, sizeof(Movie), 1, fptr); // Write the new movie to the file in place of the empty record
            printf("Movie added successfully\n"); // Inform user of success
            added = 1; // Set add to 1
        }
    }

    if (!added) { // Check if the movie was not added
        printf("No empty records available. Cannot add more movies.\n"); // Inform user no space is available
    }
}


// Function 3 to delete a movie from the file
void deleteMovie(FILE *fptr) {
    int recordNum; // Variable to store user input for record number
    printf("Enter record number to delete (1-10): "); // Prompt user for record number
    scanf("%d", &recordNum); // Read record number
    while(recordNum < 1 || recordNum > 10) { // Validate record number
        printf("Invalid record number. Please enter a number between 1 and 10: "); // Ask again if invalid
        scanf("%d", &recordNum); // Read record number again
    }
    Movie blankMovie = {"", 0, 0, 1}; // Create a blank movie record
    fseek(fptr, (recordNum-1) * sizeof(Movie), SEEK_SET); // Move to the specific record in the file
    fwrite(&blankMovie, sizeof(Movie), 1, fptr); // Overwrite the record with a blank movie
    printf("Movie deleted successfully\n"); // Inform user of success
}


// Function 4 to update a movie in the file
void updateMovie(FILE *fptr) {
    int recordNum; // Variable to store user input for record number
    printf("Enter record number to update (1-10): "); // Prompt user for record number
    scanf("%d", &recordNum); // Read record number
    while(recordNum < 1 || recordNum > 10) { // Validate record number
        printf("Invalid record number. Please enter a number between 1 and 10: "); // Ask again if invalid
        scanf("%d", &recordNum); // Read record number again
    }

    fseek(fptr, (recordNum-1) * sizeof(Movie), SEEK_SET); // Move to the specific record in the file
    Movie m; // Declare a movie structure to store the movie being updated
    fread(&m, sizeof(Movie), 1, fptr); // Read the movie from the file
    if(m.isEmpty) { // Check if the movie record is empty
        printf("That movie does not exist.\n"); // Inform user that there is no movie to update
        return;
    }

    printf("Current title: %s\n", m.title); // Display current title
    printf("Current downloads: %d\n", m.downloads); // Display current downloads

    printf("Updated title (max 20 chars): "); // Prompt user for new title
    while(getchar() != '\n'); // Clean stdin
    fgets(m.title, MAX_TITLE_LENGTH, stdin); // Read new title
    m.title[strcspn(m.title, "\n")] = 0; // Remove newline character from title

    if(strlen(m.title) >= 20) { // Check if new title is too long
        m.title[20] = '\0'; // Truncate title if too long
        printf("Title exceeds 20 characters and will be truncated\n"); // Inform user
    }

    printf("Updated number of downloads: "); // Prompt user for new number of downloads
    scanf("%d", &m.downloads); // Read new number of downloads
    printf("Updated genre (0: Action, 1: Animation, 2: Drama, 3: Sci-Fi, 4: Superhero): "); // Prompt user for new genre
    int genre; // Variable to store new genre
    scanf("%d", &genre); // Read new genre
    while(genre < 0 || genre > 4) { // Validate new genre
        printf("Invalid genre. Enter a valid genre number (0-4): "); // Ask again if invalid
        scanf("%d", &genre); // Read new genre again
    }
    m.genre = genre; // Set new genre

    fseek(fptr, -sizeof(Movie), SEEK_CUR); // Move back to the position of the record being updated
    fwrite(&m, sizeof(Movie), 1, fptr); // Write the updated movie back to the file
    printf("Movie updated successfully\n"); // Inform user of success
}


// Function 5 to display information about all movies
void displayMovieInfo(FILE *fptr) {
    fseek(fptr, 0, SEEK_SET); // Move to the start of the file
    Movie m; // Declare a movie structure to store movies read from the file
    int recordNum = 1; // Variable to number the records for display
    printf("\nNumber\tTitle\t\t\t\t\tDownloads\t\tGenre\n"); // Print table header
    printf("======  ======                  =========       =====\n"); // Print table header
    while(fread(&m, sizeof(Movie), 1, fptr)) { // Loop to read movies from the file
        if(!m.isEmpty) { // Check if the movie record is not empty
            printf("%d\t\t%-20s\t%d\t\t", recordNum, m.title, m.downloads); // Print movie details
            switch(m.genre) { // Switch statement based on genre
                case ACTION: printf("Action\n"); break; // Print genre as Action
                case ANIMATION: printf("Animation\n"); break; // Print genre as Animation
                case DRAMA: printf("Drama\n"); break; // Print genre as Drama
                case SCI_FI: printf("Sci-Fi\n"); break; // Print genre as Sci-Fi
                case SUPERHERO: printf("Superhero\n"); break; // Print genre as Superhero
            }
            recordNum++; // Increment record number
        }
    }
}


// Function 6 to display the main menu and get user choice
int displayMenu() {
    int choice; // Variable to store user's choice
    printf("\nMovie Information System\n"); // Print menu title
    printf("1. Add a movie\n"); // Menu option to add a movie
    printf("2. Delete a movie\n"); // Menu option to delete a movie
    printf("3. Update a movie\n"); // Menu option to update a movie
    printf("4. Display all movies\n"); // Menu option to display all movies
    printf("5. Exit\n"); // Menu option to exit program
    printf("\nYour choice: "); // Prompt for user's choice
    scanf("%d", &choice); // Read user's choice
    return choice;
}
```
