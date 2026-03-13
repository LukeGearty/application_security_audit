# Application Overview
A small web-based workout generator written in C and PHP. The project consists of four files:
- workout-gen.c — the core C program that reads a list of exercises from a flat file, randomly assigns sets and reps, and generates a workout.
- workout-gen-linux — the compiled Linux binary of the above.
- exercises.txt — the data file containing exercises and their associated muscle groups in CSV format.
- index.php — the web front-end that invokes the compiled binary via shell_exec and displays the output in the browser.

[Link to repository](https://github.com/LukeGearty/random_workout_generator)

## Code Quality Issues

### Incorrect Code Format
In the original code, the HTML and PHP were split incorrectly. The PHP appears after the HTML, when it should appear before or within.

Fix:
Put majority PHP above HTML and put the output within the HTML

Original web output is extremely disjointed and unorganized.
Example:
<img width="1915" height="168" alt="random_workout_original_format" src="https://github.com/user-attachments/assets/8256e198-8574-433f-964c-e51eebd552f3" />

Fix: 
Use the `<pre>` tag to fix the output

<img width="350" height="200" alt="random_workout_afterward" src="https://github.com/user-attachments/assets/87e73567-abbe-4c1f-a89e-6939920b51ba" />

### Poor Quality Outputs
The binary will output workouts that don't really make sense, such as the below:

Fix: Set a minimum number of reps and sets so the workouts don't end up being something like 1 set of 1 rep for deadlift

<img width="508" height="297" alt="random_workout_poor_workout" src="https://github.com/user-attachments/assets/bd19378d-3541-436b-bd51-1b5bc5d44a84" />

## Security Issues

### Buffer Overflow
In workout-gen.c, these lines are unsafe:

```C
strcpy(curr.name, name);
strcpy(curr.muscle_group, muscle_group);
```

`strcpy` copies until it hits a null terminator with no knowledge of the destination buffer's size. The overall buffer size for reading in the file is 100, but the destinations of name and muscle_group have buffer sizes of 50 and 30 respectively. This discrepancy, along with the unsafe function, allows a buffer overflow to occur. 

Since `strcpy` does not perform bounds checking, if a long enough line is provided, then the destination buffer will overflow and the program can crash, corrupt adjacent memory, or overwrite return addresses. 

Fix: 
Replace `strcpy` with a safer function, like `snprintf`. Snprintf provides bounds checking and guarantees null termination, which is why I used it instead of `strncpy`.

### No Bounds Checking
In the code, the following array is allocated for 50 struct exercises:
```C
Exercise* exercises = malloc(sizeof(Exercise) * MAX_EXERCISE);
```

In the while loop, there is no checking to make sure that it stays within 50. If a text file is provided with greater than 50 lines, an error will occur.

Fix: 
Track the size of `count` with an if statement and stop when it hits 50.

### Return values not Checked

In main, and in read_exercises, return values for `malloc` are not checked. Even if the memory allocation fails, the program will keep going. The return value for the `strtok` function calls are also not checked.

Fix: Add return checks such as the one below:
```C
Exercise *exercises = read_exercises(fptr, buffer, &count);

if (exercises == NULL) {
	fclose(fptr);
	return 1;
}
```
