

## 1. File Output (`ofstream`)

### Basic Setup

To use file output in C++:

1. Include the header:
    
    `#include <fstream>`
    
2. Use `ofstream` (defined in `<fstream>`) for output.
    
3. Declare an `ofstream` object (like `cout` for files).
    
4. Use `using namespace std;`
    
5. Associate the object with a file using `.open()`.
    
6. Close it with `.close()`.
    

### Example
```cpp
#include <fstream> 
#include <iostream> 
using namespace std;  
int main() 
{     
	ofstream outFile;               // declare object     
	outFile.open("fish.txt");       // open file for writing      
	double wt = 125.8;     
	outFile << wt;                  // write to file      
	char line[81] = "Objects are closer than they appear.";     
	outFile << line << endl;        // write a line of text      
	outFile.close();                // close file 
}
```

### Using User-Specified File Names

`char filename[50]; cin >> filename; ofstream fout; fout.open(filename); fout << "Some text" << endl; fout.close();`

---

## 2. File Input (`ifstream`)

### Setup

1. Include:
    
    `#include <fstream>`
    
2. Use `ifstream` for input.
    
3. Declare an input file stream object.
    
4. Use `.open()` and `.close()` as with output files.
    
5. Use `>>`, `get()`, or `getline()` for reading.
    

### Example
```cpp
#include <fstream>
#include <iostream> 
#include <cstdlib>    // for exit() using namespace std;  
int main() 
{     
	ifstream inFile;     
	inFile.open("data.txt");  // open for input      
	if (!inFile.is_open()) 
	{  // check open status         
		cerr << "Failed to open file.\n";         
		exit(EXIT_FAILURE);     
	}      
	string line;     
	while (getline(inFile, line)) 
	{         
		cout << line << endl; // read each line     
	}      
	inFile.close();           // close file 
}
```


---

## 3. Input Methods

|Method|Description|
|---|---|
|`>>`|Reads formatted data (like `cin`)|
|`get()`|Reads a single character|
|`getline()`|Reads an entire line (including spaces)|

---

## 4. Stream State Checking

You can check the **status of a file stream** using built-in methods:

|Method|Meaning|
|---|---|
|`.is_open()`|Returns `true` if the file is open|
|`.good()`|Returns `true` if everything is OK|
|`.eof()`|Returns `true` if end-of-file reached|
|`.fail()`|Returns `true` if an input/output error occurred|
- Caution: `eof()`只有在第一次读取成功以后才是true，如果什么都没有读取，它是false。
### Example Uses
```cpp
if (!inFile.is_open())     
	exit(EXIT_FAILURE);   // file failed to open  
while (inFile.good())     // continue processing  
	if (inFile.eof())     
		cout << "End of file reached.";  
	if (inFile.fail())     
		cerr << "Input type mismatch!";
```


Or more idiomatically:

`while (inFile >> value)     // loop until input fails`

---

 **Summary**

- Use `<fstream>` for both input and output.
    
- `ofstream` = output (write), `ifstream` = input (read).
    
- Always **check file state** after opening.
    
- Use `.close()` when done.
    
- Prefer idiomatic patterns like:
    
    `while (inFile >> value) { ... }`