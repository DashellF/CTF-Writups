# jailpy3

Category: **rev**

Description:

> Made with a burning passion for pyjails (i.e. creating insane payloads just to bypass some random condition), I turned everything in this python script into pyjail tech! Here's a program that's suppose to print a flag. But something seems to be getting in the way...


---

At first, this challenge looks like a simple 'find the error' challenge. We are given a python file with two lines, an import line and an ***11 megabite*** long print statement.

```
import collections
print({}.__class__.__subclasses__()[2].copy.__builtins__[{}.__class__.__subclasses__()[2].copy.__builtins__[chr(1^2^32^64)+chr(8^32^64)+chr(2^16^32^64)]({}.__class__.__subclasses__()[2].copy.__builtins__[chr(1^2^4^8^16^64)+chr(1^2^4^8^16^64)+chr(1^8^32^64)+chr(1^4^8^32^64)+chr(16^32^64)+chr(1^2^4^8^32^64)+chr(2^16^32^64)+chr(4^16^32^64)+chr(1^2^4^8^16^64)+chr(1^2^4^8^16^64)](chr(1^2^16^32^64)

...

.select.POLLRDNORM)).select.POLLRDNORM))
```

When you try and run this file, it gives you the error: 

'''
$ python3 'code.py'
Segmentation fault
'''

How this file is right now, we can't really understand what is going on.To start us off, let's start simplifying this print statement.

taking a quick glance at this file, we can see there are a lot of 'chr(1^2^32^64)' type statements, which we can simplify pretty easily. 'chr()' is a function from `builtins` that returns a character given that character's ascii table value. the statement '1^2^32^64' can be mathmatecally simplified as just adding 1+2+32+64 because all of these numbers are factors of 2 so xor can be equivalized to addition. Let's first start by going through the file and simplifying all of the xor statements.

'''
import re

def decode_chr_expressions(file_path):
    """
    Reads a Python file, decodes all chr() expressions with XOR operations
    using re.sub(), and returns the modified content.

    Args:
        file_path (str): The path to the input Python file.

    Returns:
        str: The file content with decoded characters, or None if an error occurs.
    """
    try:
        # Read the entire content of the file
        with open(file_path, 'r') as file:
            content = file.read()

        # Counter for decoded commands (debugging + its neat)
        decoded_count = 0

        def decoder_callback(match):
            nonlocal decoded_count
            expression = match.group(1)
            try:
                result = eval(expression)
                decoded_char = chr(result)
                decoded_count += 1
                return f"'{re.escape(decoded_char)}'"
            except Exception as e:
                # If there's an error, return the original string to avoid breaking the code
                return match.group(0)

        # Regular expression to find all occurrences of chr(...)
        pattern = r"chr\(([^)]+)\)"
        
        # Use re.sub() with the callback function to handle all replacements in one pass
        modified_content = re.sub(pattern, decoder_callback, content)

        return modified_content, decoded_count

    except FileNotFoundError:
        print(f"Error: The file '{file_path}' was not found.")
        return None, None
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        return None, None


decoded_content, decoded_count = decode_chr_expressions(original_file)

if decoded_content:
    # Print the summary of the process
    print("\n--- Decoding Summary ---")
    # A total count isn't as useful here as we're not iterating separately,
    # but we can print the number of successful decodes.
    print(f"Commands successfully decoded: {decoded_count}")
    
    # Save the decoded content to a new file named 'decoded1.py' (you can edit this to be the same file or if you want a new one with a different name)
    try:
        with open('decoded1.py', 'w') as new_file:
            new_file.write(decoded_content)
        print("\nDecoded content successfully saved to 'decoded1.py'")
    except Exception as e:
        print(f"Error writing to file 'decoded1.py': {e}")
'''

With that, you are left with a smaller file 'decoded1.py' that now looks like this

```
import collections
print({}.__class__.__subclasses__()[2].copy.__builtins__[{}.__class__.__subclasses__()[2].copy.__builtins__['c'+'h'+'r']({}.__class__.__subclasses__()[2].copy.__builtins__['_'+'_'+'i'+'m'+'p'+'o'+'r'+'t'+'_'+'_']('s'+'u'+'b'+'p'+'r'+'o'+'c'+'e'+'s'+'s').select.POLLIN^{}.__class__.__subclasses__()[2].copy.__builtins__['_'+'_'+'i'+'m'+'p'+'o'+'r'+'t'+'_'+'_']

...

.select.POLLRDNORM)).select.POLLRDNORM))
```

Next, to make this more readable, let's get rid of all of the '+' signs and combine all of the repeatedly added characters into strings. You can do this several ways, here is what I did:


```
import os
import re

def simplify_concatenated_strings(file_path):
    """
    Reads a file, finds concatenated string expressions like "'c'+'h'+'r'",
    and replaces them with the single resulting string.

    Args:
        file_path (str): The path to the input Python file.

    Returns:
        tuple: A tuple containing the modified content (str) and a count of
               replacements made (int). Returns (None, None) if an error occurs.
    """
    try:
        if not os.path.exists(file_path):
            print(f"Error: The file '{file_path}' was not found.")
            return None, None

        with open(file_path, 'r') as file:
            content = file.read()
        
        replacements_made = 0
        
        # This regex finds a single-quoted string followed by a plus sign and another single-quoted string.
        # It handles multiple concatenations in a single expression.
        pattern = re.compile(r"(\'[^\']*?\'\s*\+\s*\'[^\']*?\')(?:\s*\+\s*\'[^\']*?\')*")
        
        def replacer_callback(match):
            nonlocal replacements_made
            expression = match.group(0)
            
            try:
                # Use eval() to combine the strings.
                simplified_string = eval(expression)
                replacements_made += 1
                
                # Return the new single-quoted string.
                return f"'{simplified_string}'"
            except Exception as e:
                print(f"Error evaluating string expression '{expression}': {e}")
                return match.group(0)

        # Perform the replacements
        simplified_content = re.sub(pattern, replacer_callback, content)
        
        return simplified_content, replacements_made
    
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        return None, None

# Specify the input and output file paths
source_file = 'decoded1.py'
output_file = 'decoded2.py'

# Run the replacement function
simplified_content, count = simplify_concatenated_strings(source_file)

if simplified_content:
    print("\n--- Replacement Summary ---")
    print(f"Successfully simplified {count} concatenated string expressions.")
    
    try:
        with open(output_file, 'w') as new_file:
            new_file.write(simplified_content)
        print(f"\nSimplified content successfully saved to '{output_file}'")
    except Exception as e:
        print(f"Error writing to file '{output_file}': {e}")
```

With that, our file now will look like this

```
import collections
print({}.__class__.__subclasses__()[2].copy.__builtins__[{}.__class__.__subclasses__()[2].copy.__builtins__['chr']({}.__class__.__subclasses__()[2].copy.__builtins__['__import__']('subprocess').select.POLLIN^{}

...

.select.POLLRDNORM)).select.POLLRDNORM))

Now it gets a little more complex to simplify. We see a repeated call of `{}.__class__.__subclasses__()[2].copy.__builtins__`. In short, this is just a call to `__builtins__`. We can simply replace all instances of `{}.__class__.__subclasses__()[2].copy.__builtins__` with `__builtins__`. You might think to do this with a find-and-replace tool on a file editing software, but these tools often break or skip instances with large files.

```
import re

def simplify_builtins_expressions(file_path):
    """
    Reads a file and replaces the obfuscated __builtins__ access
    with the simple '__builtins__' name.

    Args:
        file_path (str): The path to the input Python file.

    Returns:
        tuple: A tuple containing the modified content (str) and a count of
               replacements made (int). Returns (None, None) if an error occurs.
    """
    try:
        with open(file_path, 'r') as file:
            content = file.read()

        # Regex pattern to find the obfuscated '__builtins__' access.
        # This is the exact repeating string from your example.
        pattern = r"\{\}\.__class__\.__subclasses__\(\)\[2\]\.copy\.__builtins__"
        
        # Use re.sub() with a count parameter to perform all replacements and
        # get a count of how many were made.
        simplified_content, count = re.subn(pattern, '__builtins__', content)
        
        return simplified_content, count
        
    except FileNotFoundError:
        print(f"Error: The file '{file_path}' was not found.")
        return None, None
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        return None, None

# Run the script on the output from your previous step.
source_file = 'decoded2.py'
simplified_content, count = simplify_builtins_expressions(source_file)

if simplified_content:
    print("\n--- Simplification Summary ---")
    print(f"Instances of '__builtins__' expression simplified: {count}")
    
    # Save the new, cleaner code to a file.
    try:
        with open('decoded3.py', 'w') as new_file:
            new_file.write(simplified_content)
        print("\nSimplified content successfully saved to 'decoded3.py'")
    except Exception as e:
        print(f"Error writing to file 'decoded3.py': {e}")
```


that makes things much more readable,

```
import collections
print(__builtins__[__builtins__['chr'](__builtins__['__import__']('subprocess').select.POLLIN^__builtins__['__import__']('subprocess').select.POLLPRI^__builtins__['__import__']('subprocess').select.POLLNVAL^__builtins__['__import__']('subprocess').select.POLLRDNORM)+__builtins__['chr'](__builtins__['__import__']('subprocess').select.POLLERR^__builtins__['__import__']('subprocess').select.POLLNVAL^__builtins__['__import__']('subprocess').select.POLLRDNORM)+__builtins__['chr'](__builtins__['__import__']('subprocess').select.POLLPRI^__builtins__['__import__']

...

.select.POLLRDNORM)).select.POLLRDNORM))
```

after this process, we can see a bunch of the __builtins__['chr'] commands, we already wrote a script that replaces them, so lets use it again! Make sure to update the input and output files.

that leaves us with

```
print(__builtins__[chr(__builtins__['__import__']('subprocess').select.POLLIN^__builtins__['__import__']('subprocess').select.POLLPRI^__builtins__['__import__']('subprocess').select.POLLNVAL^__builtins__['__import__']('subprocess').select.POLLRDNORM)+chr(__builtins__['__import__']('subprocess').select.POLLERR^__builtins__['__import__']

...

.select.POLLRDNORM)).select.POLLRDNORM))
```


From here, we need to look into the `__builtins__['__import__']('subprocess').select.POLLIN` type lines. They repeat a lot, just with different ending calls. A quick google search tells us that this code is supposed to simply return whatever number `POLLIN` is set to in the subprocess function. Thus, we can replace this whole line with the integer `1` because that is what POLLIN is in subprocess. Another google search tells us what other values are set to:

POLLIN: 1,
POLLPRI: 2,
POLLOUT: 4,
POLLRDNORM: 64,
POLLRDBAND: 128,
POLLWRNORM: 256,
POLLWRBAND: 512,
POLLERR: 8,
POLLHUP: 16,
POLLNVAL: 32,

With this information, we can greatly simplify what is left. Replace all of the `__builtins__['__import__']('subprocess').select.POLLIN` calls with the corresponding integer.

```
import os
import re

def replace_obfuscated_select_constants(file_path):
    """
    Reads a file and replaces specific obfuscated select module constant
    strings with their integer values.

    Args:
        file_path (str): The path to the input Python file.

    Returns:
        tuple: A tuple containing the modified content (str) and a count of
               replacements made (int). Returns (None, None) if an error occurs.
    """
    try:
        if not os.path.exists(file_path):
            print(f"Error: The file '{file_path}' was not found.")
            return None, None

        with open(file_path, 'r') as file:
            content = file.read()
            
        replacements_made = 0
        
        # A mapping of constant names to their integer values
        constant_values = {
            'POLLIN': 1,
            'POLLPRI': 2,
            'POLLOUT': 4,
            'POLLRDNORM': 64,
            'POLLRDBAND': 128,
            'POLLWRNORM': 256,
            'POLLWRBAND': 512,
            'POLLERR': 8,
            'POLLHUP': 16,
            'POLLNVAL': 32,
        }
        
        # A regular expression to find the entire obfuscated string and capture the constant name.
        pattern = re.compile(
            r"__builtins__\['__import__']\('subprocess'\)\.select\.([A-Z_]+)"
        )
        
        def replacer_callback(match):
            nonlocal replacements_made
            constant_name = match.group(1) # This is the captured constant name (e.g., 'POLLIN')
            
            # Look up the integer value from our dictionary
            if constant_name in constant_values:
                replacements_made += 1
                return str(constant_values[constant_name])
            else:
                # If we don't recognize the constant, return the original string
                return match.group(0)
            
        simplified_content = re.sub(pattern, replacer_callback, content)
        
        # Add the necessary imports to the top of the file
        new_imports = (
            "import subprocess\n"
            "from select import *\n"
            "import collections\n"
            "import builtins\n\n"
        )
        
        final_content = new_imports + simplified_content

        return final_content, replacements_made
    
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        return None, None

# Specify the input and output file paths
source_file = 'decoded4.py' # Assuming the previous file was decoded4.py
output_file = 'decoded5.py'

# Run the replacement function
simplified_content, count = replace_obfuscated_select_constants(source_file)

if simplified_content:
    print("\n--- Replacement Summary ---")
    print(f"Successfully replaced {count} obfuscated constants with their integer values.")
    
    try:
        with open(output_file, 'w') as new_file:
            new_file.write(simplified_content)
        print(f"\nSimplified content successfully saved to '{output_file}'")
    except Exception as e:
        print(f"Error writing to file '{output_file}': {e}")
```


That last simplification greatly reduced the size of the file. Now, the file is only about 
102 kb. We have a very familiar look to the print statement now:

```
print(__builtins__[chr(1^2^32^64)+chr(8^32^64)+chr(2^16^32^64)](__builtins__[chr(1^2^4^8^16^64)+chr(1^2^4^8^16^64)+chr(1^8^32^64)+chr(1^4^8^32^64)+chr(16^32^64)+chr(1^2^4^8^32^64)+chr(2^16^32^64)+chr(4^16^32^64)+chr(1^2^4^8^16^64)+ 

...

)

```
We already wrote code to simplify the xor statements, so lets do that first. Remember to change the input and output files in the code!

Once we run that, we get a line also looking quite familiar
```
print(__builtins__['c'+'h'+'r'](__builtins__['_'+'_'+'i'+'m'+'p'+'o'+'r'+'t'+'_'+'_']('s'+'u'+'b'+'p'+'r'+'o'+'c'+'e'+'s'+'s').select.

...

)
```

Lets run our substring connection script again on this file.

Once we do that, we get a file looking like a lot of these statements

```
__builtins__['chr'](__builtins__['__import__']('subprocess').select.POLLPRI
...

First, let's run our script that replaces `__builtins__['chr']` with `chr` (at this point, you can probably just use a find and replace tool in whatever file editor you want).

Then, lets run our script that simplifies `__builtins__['__import__']('subprocess').select.POLLPRI` statements again.

With both of those scripts run again, your decoded python file should look like this:

```
print(chr(4^8^64)+chr(1^8^64)+chr(4^16^64)+chr(1^2^64)+chr(4^16^64)+chr(2^4^64)+chr(1^2^8^16^32^64)+chr(8^32^64)+chr(16^32)+chr(1^2^4^16^32^64)+chr(1^2^4^8^16^64)+chr(1^2^32^64)+chr(16^32)+chr(2^4^8^32^64)+chr(2^4^16^32^64)+chr(1^2^4^8^32^64)+chr(4^8^32^64)

...

)
```

Now that we are back to chr() statements, let's run our script to simplify these commands.

once that is done, you get this file:
```
import collections
print('L'+'I'+'T'+'C'+'T'+'F'+'{'+'h'+'0'+'w'+'_'+'c'+'0'+'n'+'v'+'o'+'l'+'u'+'7'+'e'+'d'+'_'+'c'+'4'+'n'+'_'+'i'+'7'+'_'+'g'+__builtins__['__import__']('types').FunctionType(__builtins__['__import__']('marshal').loads(__builtins__['bytes'].fromhex('630000000000000000000000000300000000000000f3300000009700640064016c005a00020065006a020000000000000000000000000000000000006402ab01000000000000010079012903e9000000004ee9010000002902da026f73da055f65786974a900f300000000fa033c783efa083c6d6f64756c653e7208000000010000007314000000f003010101db0009883888328f38893890418d3b7206000000')), {'os': __builtins__['__import__']('os')})()+'3'+'7'+'_'+'f'+'0'+'r'+'_'+'0'+'n'+'3'+'_'+'s'+'1'+'m'+'p'+'l'+'3'+'_'+'w'+'0'+'r'+'k'+'4'+'r'+'o'+'u'+'n'+'d'+'?'+'?'+'}')
```

From here, you can probably read and get the flag, but to make the flag easier to copy, let's finish this off with our script that combines characters.

With that, we get the file:
```
import collections
print('LITCTF{h0w_c0nvolu7ed_c4n_i7_g'+__builtins__['__import__']('types').FunctionType(__builtins__['__import__']('marshal').loads(__builtins__['bytes'].fromhex('630000000000000000000000000300000000000000f3300000009700640064016c005a00020065006a020000000000000000000000000000000000006402ab01000000000000010079012903e9000000004ee9010000002902da026f73da055f65786974a900f300000000fa033c783efa083c6d6f64756c653e7208000000010000007314000000f003010101db0009883888328f38893890418d3b7206000000')), {'os': __builtins__['__import__']('os')})()+'37_f0r_0n3_s1mpl3_w0rk4round??}')
```

Leaving us with the flag:
`LITCTF{h0w_c0nvolu7ed_c4n_i7_g37_f0r_0n3_s1mpl3_w0rk4round??}`


If we continue to simplify this code, we can also see what was crashing the code.
The code that divides the flag simplifies down to the line `os._exit(2)` which simply exits the code prematurely.