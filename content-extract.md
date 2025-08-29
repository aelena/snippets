# About

- Fetches HTML content from any accessible URL
- Finds all elements with a specified CSS class name using BeautifulSoup
- Extracts and cleans the text content from those elements
- Concatenates all text pieces with double newlines for readability
- Saves the output to a text file with UTF-8 encoding

__Basic handling of__

- HTTP errors and network issues
- Empty or missing elements
- Text cleaning (strips whitespace)
- UTF-8 encoding for international characters
- User-agent header to avoid basic bot blocking

## Dependencies

`pip install requests beautifulsoup4`

## Usage 
`python script.py https://example.com article-content output.txt`

# Source

```python
import requests
from bs4 import BeautifulSoup
import sys

def extract_class_text(url, class_name, output_file="extracted_text.txt"):
    """
    Extract text content from all HTML elements with specified class name
    and save to a text file.
    
    Args:
        url (str): The URL to scrapels
        class_name (str): The CSS class name to search for
        output_file (str): Output filename (default: "extracted_text.txt")
    """
    try:
        # Send GET request to the URL
        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
        }
        response = requests.get(url, headers=headers)
        response.raise_for_status()  # Raise an exception for bad status codes
        print(response)
        # Parse the HTML content
        soup = BeautifulSoup(response.content, 'html.parser')
        
        # Find all elements with the specified class
        elements = soup.find_all(class_=class_name)
        # print(elements.__len__)
        if not elements:
            print(f"No elements found with class '{class_name}'")
            return

        
        # Extract and concatenate text content
        extracted_texts = []
        for element in elements:
            text = element.get_text(strip=True)
            print(text)
            if text:  # Only add non-empty text
                extracted_texts.append(text)
        
        # Write to file
        with open(output_file, 'w', encoding='utf-8') as f:
            f.write('\n\n'.join(extracted_texts))
        
        print(f"Successfully extracted text from {len(extracted_texts)} elements")
        print(f"Output saved to: {output_file}")
        
    except requests.exceptions.RequestException as e:
        print(f"Error fetching URL: {e}")
    except Exception as e:
        print(f"An error occurred: {e}")

def main():
    """Main function to handle command line usage"""
    if len(sys.argv) < 3:
        print("Usage: python script.py <url> <class_name> [output_file]")
        print("Example: python script.py https://example.com article-content output.txt")
        return
    
    url = sys.argv[1]
    class_name = sys.argv[2]
    print(url)
    print(class_name)
    output_file = sys.argv[3] if len(sys.argv) > 3 else "extracted_text.txt"
    
    extract_class_text(url, class_name, output_file)

if __name__ == "__main__":
    # Example usage (uncomment to test)
    # extract_class_text("https://example.com", "article-content", "output.txt")
    
    # Run main function for command line usage
    main()
```
