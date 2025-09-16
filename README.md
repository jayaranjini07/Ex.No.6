# Ex.No.6 Development of Python Code Compatible with Multiple AI Tools

# Date: 16-09-2025
# Register no: 212223060101
# Aim:
Write and implement Python code that integrates with multiple AI tools to automate the task of interacting with APIs, comparing outputs, and generating actionable insights with Multiple AI Tools

# AI Tools Required:
- Chatgpt
- Claude
- Gemini
# Explanation:
Experiment the persona pattern as a programmer for any specific applications related with your interesting area. 
Generate the outoput using more than one AI tool and based on the code generation analyse and discussing that. 

# Conclusion:

## Prompt:
" You are a skilled Python programmer. Write a single Python function that can automatically read both .csv and .json files into a list of dictionaries. Use only standard Python libraries (such as csv, json, and os). Make sure the code is clean, well-commented, and handles invalid file types gracefully. After writing the code, explain your approach step by step."

This prompt clearly:
- Sets the persona (skilled Python programmer)
- Defines the task (one function for .csv and .json)
- Constrains the tools (only standard libraries)
- Adds quality requirements (clean, commented, error-handling)
- Asks for an explanation (for analysis in your experiment)

## Respones:

### Chatgpt
```
import os
import csv
import json
from typing import List, Dict, Any

def read_file_to_records(path: str, encoding: str = "utf-8") -> List[Dict[str, Any]]:
    """
    Read a .csv or .json file and return a list of dictionaries (records).

    Behavior:
      - .csv -> uses csv.DictReader; trims header/value whitespace and strips BOM if present.
      - .json -> accepts:
          * a list of dicts -> returned as-is (non-dict elements are wrapped as {'value': ...})
          * a single dict -> returned as [that_dict]
          * a dict-of-lists with all lists of equal length -> converted to list-of-row-dicts (transpose)
      - Unsupported extension -> raises ValueError with a clear message.
      - File not found -> raises FileNotFoundError.

    Args:
        path: Path to the .csv or .json file.
        encoding: File encoding (default "utf-8").

    Returns:
        List of dictionaries representing rows/records.

    Raises:
        FileNotFoundError, ValueError (for unsupported extensions or parse errors).
    """
    if not os.path.isfile(path):
        raise FileNotFoundError(f"File not found: {path!r}")

    _, ext = os.path.splitext(path)
    ext = ext.lower()

    if ext == ".csv":
        try:
            with open(path, "r", encoding=encoding, newline="") as f:
                reader = csv.DictReader(f)
                records: List[Dict[str, Any]] = []
                for row in reader:
                    # Clean keys and values: strip whitespace; also remove BOM from keys if present
                    cleaned = {}
                    for k, v in row.items():
                        if k is None:
                            # shouldn't normally occur, but guard anyway
                            continue
                        clean_key = k.lstrip("\ufeff").strip()
                        clean_val = v.strip() if isinstance(v, str) else v
                        cleaned[clean_key] = clean_val
                    records.append(cleaned)
                return records
        except csv.Error as e:
            raise ValueError(f"Error parsing CSV file {path!r}: {e}") from e

    elif ext == ".json":
        try:
            with open(path, "r", encoding=encoding) as f:
                data = json.load(f)
        except json.JSONDecodeError as e:
            raise ValueError(f"Error decoding JSON file {path!r}: {e}") from e

        # If the JSON is a list -> convert elements to dicts where needed
        if isinstance(data, list):
            records: List[Dict[str, Any]] = []
            for item in data:
                if isinstance(item, dict):
                    records.append(item)
                else:
                    # non-dict items get wrapped so function always returns list[dict]
                    records.append({"value": item})
            return records

        # If the JSON is a dict -> try to detect dict-of-lists (table-like) and transpose
        if isinstance(data, dict):
            # Collect entries that are lists
            list_items = {k: v for k, v in data.items() if isinstance(v, list)}
            # If every top-level value is a list and all lists have the same length -> transpose
            if list_items and len(list_items) == len(data):
                lengths = [len(v) for v in list_items.values()]
                if lengths and all(l == lengths[0] for l in lengths):
                    n = lengths[0]
                    records = []
                    for i in range(n):
                        row = {k: data[k][i] for k in data.keys()}
                        records.append(row)
                    return records
            # Fallback: wrap single dict into a list
            return [data]

    else:
        raise ValueError(
            f"Unsupported file extension {ext!r}. Supported extensions: '.csv', '.json'."
        )
```
### Claude
```
import csv
import json
import os


def read_file_to_dict_list(file_path):
    """
    Reads CSV or JSON files and returns data as a list of dictionaries.
    
    Args:
        file_path (str): Path to the file to be read
        
    Returns:
        list: List of dictionaries containing the file data
        
    Raises:
        FileNotFoundError: If the file doesn't exist
        ValueError: If the file type is not supported or file format is invalid
        Exception: For other file reading errors
    """
    
    # Check if file exists
    if not os.path.exists(file_path):
        raise FileNotFoundError(f"File not found: {file_path}")
    
    # Get file extension to determine file type
    _, file_extension = os.path.splitext(file_path.lower())
    
    try:
        if file_extension == '.csv':
            return _read_csv_file(file_path)
        elif file_extension == '.json':
            return _read_json_file(file_path)
        else:
            raise ValueError(f"Unsupported file type: {file_extension}. Only .csv and .json files are supported.")
            
    except (csv.Error, json.JSONDecodeError) as e:
        raise ValueError(f"Invalid file format: {str(e)}")
    except Exception as e:
        raise Exception(f"Error reading file: {str(e)}")


def _read_csv_file(file_path):
    """
    Helper function to read CSV files.
    
    Args:
        file_path (str): Path to the CSV file
        
    Returns:
        list: List of dictionaries with CSV data
    """
    data = []
    
    with open(file_path, 'r', encoding='utf-8', newline='') as csvfile:
        # Use Sniffer to detect CSV dialect (delimiter, quote char, etc.)
        sample = csvfile.read(1024)
        csvfile.seek(0)
        
        try:
            dialect = csv.Sniffer().sniff(sample)
            reader = csv.DictReader(csvfile, dialect=dialect)
        except csv.Error:
            # Fallback to default dialect if sniffing fails
            csvfile.seek(0)
            reader = csv.DictReader(csvfile)
        
        # Read all rows into list of dictionaries
        for row in reader:
            # Strip whitespace from keys and values for cleaner data
            clean_row = {key.strip(): value.strip() if isinstance(value, str) else value 
                        for key, value in row.items()}
            data.append(clean_row)
    
    return data


def _read_json_file(file_path):
    """
    Helper function to read JSON files.
    
    Args:
        file_path (str): Path to the JSON file
        
    Returns:
        list: List of dictionaries with JSON data
    """
    with open(file_path, 'r', encoding='utf-8') as jsonfile:
        json_data = json.load(jsonfile)
    
    # Handle different JSON structures
    if isinstance(json_data, list):
        # JSON is already a list - ensure all items are dictionaries
        if all(isinstance(item, dict) for item in json_data):
            return json_data
        else:
            raise ValueError("JSON list contains non-dictionary items")
    elif isinstance(json_data, dict):
        # JSON is a single dictionary - wrap in list
        return [json_data]
    else:
        raise ValueError("JSON must contain either a dictionary or a list of dictionaries")


# Example usage and testing
if __name__ == "__main__":
    # Example usage
    try:
        # Reading a CSV file
        csv_data = read_file_to_dict_list("example.csv")
        print("CSV Data:")
        print(csv_data[:2])  # Print first 2 rows
        
        # Reading a JSON file
        json_data = read_file_to_dict_list("example.json")
        print("\nJSON Data:")
        print(json_data[:2])  # Print first 2 items
        
    except FileNotFoundError as e:
        print(f"File error: {e}")
    except ValueError as e:
        print(f"Format error: {e}")
    except Exception as e:
        print(f"Unexpected error: {e}")
```
### Gemini
```
import csv
import json
import os

def read_data_to_dicts(file_path):
    """
    Reads data from a .csv or .json file into a list of dictionaries.

    Args:
        file_path (str): The path to the input file.

    Returns:
        list: A list of dictionaries, where each dictionary represents a row/object.
              Returns an empty list if the file is not found or the format is invalid.
    """
    # Check if the file exists
    if not os.path.exists(file_path):
        print(f"Error: File not found at '{file_path}'")
        return []

    # Get the file extension
    file_extension = os.path.splitext(file_path)[1].lower()

    # Handle .csv files
    if file_extension == '.csv':
        try:
            with open(file_path, mode='r', newline='', encoding='utf-8') as file:
                # Use csv.DictReader to automatically read rows as dictionaries
                csv_reader = csv.DictReader(file)
                return list(csv_reader)
        except Exception as e:
            print(f"Error reading CSV file: {e}")
            return []

    # Handle .json files
    elif file_extension == '.json':
        try:
            with open(file_path, mode='r', encoding='utf-8') as file:
                data = json.load(file)
                # Ensure the loaded data is a list of dictionaries
                if isinstance(data, list) and all(isinstance(item, dict) for item in data):
                    return data
                else:
                    print("Error: JSON file does not contain a list of dictionaries.")
                    return []
        except Exception as e:
            print(f"Error reading JSON file: {e}")
            return []

    # Handle unsupported file types
    else:
        print(f"Error: Unsupported file type '{file_extension}'. Only .csv and .json are supported.")
        return []
```
## 📊 Comparison of AI Outputs
| **Aspect**                                | **Claude’s Response**                                                                                                            | **ChatGPT’s Response**                                                                                                                               | **Gemini’s Response**                                                                       |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **Code Style**                            | Modular: main function + helper functions (`_read_csv_file`, `_read_json_file`).                                                 | Monolithic but structured: single function handling multiple formats. Uses detailed logic.                                                           | Simple and flat: one function, no helpers.                                                  |
| **Error Handling**                        | Strong error handling with custom exceptions (`FileNotFoundError`, `ValueError`, generic `Exception`). Sniffer fallback for CSV. | Robust error handling: catches `csv.Error`, `JSONDecodeError`, checks for dict-of-lists, unsupported extension.                                      | Minimal error handling: prints errors and returns empty list instead of raising exceptions. |
| **CSV Handling**                          | Uses `csv.Sniffer()` to auto-detect dialect; strips whitespace from keys/values.                                                 | Uses `csv.DictReader`; strips BOM from keys, trims whitespace.                                                                                       | Uses `csv.DictReader`; no advanced cleaning or dialect detection.                           |
| **JSON Handling**                         | Accepts dict, list of dicts; validates that list items are dicts; raises errors otherwise.                                       | Handles multiple cases: list of dicts, list with mixed types (wraps non-dicts), dict-of-lists (table transpose), or single dict. Very comprehensive. | Only accepts list of dicts; rejects other formats.                                          |
| **Documentation**                         | Clear docstrings for main and helper functions.                                                                                  | Extensive docstring with explanation of behavior for different cases.                                                                                | Concise docstring, explains basic usage.                                                    |
| **Output Consistency**                    | Always returns list of dicts (validated).                                                                                        | Always returns list of dicts (converted/wrapped if needed).                                                                                          | Returns list of dicts only if conditions met; otherwise returns `[]`.                       |
| **Persona Alignment (Python Programmer)** | Professional, modular, production-ready style.                                                                                   | Advanced developer style with detailed edge case handling.                                                                                           | Beginner-friendly, straightforward, minimalistic.                                           |
## 🔎 Insights

- Claude → Focuses on robust modular design, with helpers and clean separation of concerns. Good for production-quality maintainable code.
- ChatGPT → Most comprehensive and intelligent handling of edge cases (like dict-of-lists in JSON). Feels like an experienced engineer anticipating unusual input structures.
- Gemini → Simplest solution, easier for beginners to follow. But trades off robustness (returns empty list instead of raising exceptions).
  
# Result: 
The corresponding Prompt is executed successfully.
