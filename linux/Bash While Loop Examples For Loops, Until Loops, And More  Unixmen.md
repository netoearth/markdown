[![codes](https://www.unixmen.com/wp-content/uploads/2022/08/pexels-negative-space-92904-1-696x464.jpg "Bash While Loop Examples: For Loops, Until Loops, And More")](https://www.unixmen.com/wp-content/uploads/2022/08/pexels-negative-space-92904-1-scaled.jpg)

Bash programming comprises three kinds of loops: the for loop, the while loop, and the until loop. The idea of all the loops is to repeatedly execute some code as long as some specific parameters are met. 

The Bash for loop operates differently from the for loops in other programming languages you may be familiar with. With it, you can iterate over a series of “words” in a string. 

On the other hand, the while loop executes the code under it if the control expression (or simply, the “condition”) is true. The loop stops executing the code when the condition becomes false, or there is an explicit break in the code.

The until loop is comparable to the while loop, with the only difference being that the code keeps executing while the control expression is deemed false.

In this post, we discuss how to use the Bash while loop with examples.

## **Understanding the While Loop Syntax**

Take a look at the simple syntax of the while loop:

while \[ condition \] do  
    commands done

  
The beginning and end of a while loop are defined by the do and done keywords, respectively. The execution/termination condition is defined inside the square brackets at the beginning of the loop.

To run the examples below, create a Bash file with the .sh extension and paste the code inside it. You can then run the code by typing “bash” followed by the filename (for example, “whileloop.sh”) in the terminal and hitting enter. 

### **#1 Iterating a Fixed Number of Times**

#!/bin/bash

\# Initializing a counter variable  
counter=1  
\# Making a while loop while \[ $counter -le 3 \]  
do  
    # Printing the counter value in every iteration     echo “Iteration count: $counter”    # Incrementing the value     (( counter++ ))Done

  
The above while loop iterates over the code block three times and prints the statement every time. Running the script will generate the following outcome:

Iteration count: 1  
Iteration count: 2  
Iteration count: 3

### **#2 Using the “Break” Statement**

The break statement allows you to terminate the loop if a specific condition is met. Create a Bash file with the following code:

#!/bin/bash

\# Initializing a counter variable  
counter=1  
\# Making a loop while \[ $counter -le 5 \]  
do  
    # Checking the counter value    if \[ $counter == 3 \]    then         echo “Ending the loop”               break    fi    # Printing the counter value   echo “Iteration number $counter”    # Incrementing the value    (( counter++ ))Done

The above while loop iterates over the code three times, but the counter value is only printed two times since the break statement terminates the loop when the counter reaches value “3.” Running the script will generate the following outcome:

Iteration number 1  
Iteration number 2  
Ending the loop

### **#3 Removing a Step with the Continue Statement**

#!/bin/bash

\# Initializing the counter  
counter=0  
\# Iterate the loop four times  
while \[ $counter -le 4 \]  
do  
    # Increment counter value    (( counter++ ))

      # Check counter value    if \[ $counter == 3 \]    then         continue    fi    # Printing counter value    echo ” Iteration number $counter”

done

  
The above code will iterate through the loop four times, but when the counter value reaches three, the continue value skips the rest of the code, jumps back to the beginning, and checks the condition.

The output for the code is:

Iteration number 1  
Iteration number 2  
Iteration number 4  
Iteration number 5

### **#4 Reading a File**

#!/bin/bash

\# Checking whether a command-line argument is provided if \[ $# -gt 0 \]; then 

    # Assigning the filename from the provided value    filename=$1

    # Reading the file    while read line; do        # Printing every line        echo $line    done < $filenameelse  
    # If there is no argument in the command line, printing a message    echo “No argument provided.”fi

  
Assuming that you paste this code in a file named “ReadingFileWithWhile.sh,” running the command “bash ReadingFileWithWhile.bash” will return “No argument provided.”

However, if you run the same command and pass a text file to it, the output will look like this:

$ bash ReadingFileWithWhile.bash names.txt  
Name1  
Name2  
Name3

### **#5 Writing Text into a File**

The following code will ask for a filename from the user and write the provided text to it:

#! /bin/bash

echo -n “Enter the name of the file you want to create: “  
\# Reading the provided name  
read filename  
\# Reading the text entered into the terminal  
while read line  
do  
    echo $line >> $filenamedone

  
To end the loop reading the content, you must press Ctrl+D.

### **#6 Creating an Infinite Loop**

Writing an infinite loop can be a suitable solution for many problems. Making an infinite while loop is as simple as not mentioning a termination condition. However, you will need to use an exit statement to stop the loop as necessary. 

The following loop will iterate six times, and when the value becomes six, the exit statement will stop the loop:

#!/bin/bash

\# Initialize the counter  
counter=1  
\# Make the infinite loop  
while :  
do  
    printf “Iteration number: $counter\\n”    if \[ $counter == 2 \]    then        echo “level 1”    elif \[ $counter == 4 \]    then        echo “level 2”    elif \[ $counter == 6 \]     then         echo “level 3”        exit 0    fi      # Increment counter    ((counter++))

done

  
The output of the script looks like this:

Iteration number: 1  
Iteration number: 2  
level 1  
Iteration number: 3  
Iteration number: 4  
level 2  
Iteration number: 5  
Iteration number: 6  
level 3

## **Conclusion**

In this guide, we discuss a handful of the different ways you can use while loops in Bash. There are other ways to use the while loop, and you can solve many different problems with it.

Now that you know the fundamentals of the while loop, you can implement it in your Bash scripts.