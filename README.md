import os
import json
import sys

class TerminalColors:
    OKGREEN = '\033[92m'
    WARNING = '\033[93m'
    FAIL = '\033[91m'
    ENDC = '\033[0m'

um_soup_ascii = r'''
    ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⡟⢀⣴⠂⣠⠆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
    ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢰⡟⢀⡾⠃⣴⠏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
    ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⣄⠘⣧⡀⢿⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
    ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⣦⠈⢷⡄⠹⡆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
    ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠟⠀⠼⠃⠰⠇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
    ⠀⠀⢰⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⣶⡆⠀⠀
    ⠀⠀⠀⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠀⠀⠀
    ⠀⠀⠀⠈⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠁⠀⠀⠀
    ⠀⠀⠀⠀⠀⠙⠻⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠟⠋⠀⠀⠀⠀⠀
    ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢉⣉⣉⣉⣉⣉⣉⡉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
    ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⠉⠉⠉⠉⠉⠁⠀⠀
[um-soup] - FiveM Clothes File Finder
[uyuyorum-team] - uyuyorum ~ sequester
[discord] - discord.gg/uyuyorum
'''

print(f"{TerminalColors.WARNING}{um_soup_ascii}{TerminalColors.ENDC}")

prompt_text = f"{TerminalColors.WARNING}Please enter the clothes folder [stream] path:{TerminalColors.ENDC} "
folder_path = input(prompt_text)

directory = r'{}'.format(folder_path)

if not os.path.basename(directory) == 'stream':
    print(f"{TerminalColors.FAIL}Please specify only the [stream] folder of your clothes folder{TerminalColors.ENDC}")
    sys.exit()

# [TR] Kök adını dosya adından çıkarır
# [EN] Removes the root name from the file name
def extract_root_name(filename):
    return filename.split('^')[0]

# [TR] Kategori adını dosya adından çıkarır ve gerektiğinde 'p_' önekini kaldırır
# [EN] Extracts the category name from the file name and removes 'p_' prefix if necessary
def extract_category(filename):
    parts = filename.split('^')[1].split('_')
    category = parts[0]
    if category.startswith('p'):
        category = parts[1]
    return category.upper()

def extract_item_number(filename):
    # [TR] Öğe numarasını dosya adından çıkarır ve başındaki sıfırları kaldırır
    # [EN] Extracts the item number from the file name and removes leading zeros
    parts = filename.split('^')[1].split('_')
    
    if len(parts) < 4:
        return "0"

    number = parts[3] if parts[2].startswith('diff') else parts[2]
    if number.startswith('0') and not number.startswith('000'):
        return number.lstrip('0')
    elif number.startswith('000'):
        return '0'
    else:
        return number

# [TR] Texture harfini dosya adından çıkarır
# [EN] Extracts the texture letter from the file name
def extract_texture_letter(filename):
    parts = filename.split('^')[1].split('_')
    diff_str = None
    for part in parts:
        if part.startswith('diff'):
            diff_str = part
            break
    if diff_str is not None:
        return parts[parts.index(diff_str) + 2]
    else:
        return None

clothes_dict = {}

# Gumamit ng os.walk() para matagos ang lahat ng subfolder
total_files = 0
for dirpath, dirnames, filenames in os.walk(directory):
    print(f"Processing folder: {dirpath}")
    for filename in filenames:
        if ('.ydd' in filename or '.ytd' in filename) and '^' in filename:
            total_files += 1
            root_name = extract_root_name(filename)
            category = extract_category(filename)

            if root_name not in clothes_dict:
                clothes_dict[root_name] = {}
            if category not in clothes_dict[root_name]:
                clothes_dict[root_name][category] = {}

            item_number = extract_item_number(filename)

            if filename.endswith('.ydd'):
                if item_number not in clothes_dict[root_name][category] and not item_number.endswith('.ydd'):
                    clothes_dict[root_name][category][item_number] = {"textures": []}

            if filename.endswith('.ytd'):
                texture_letter = extract_texture_letter(filename)
                if texture_letter is not None and texture_letter.endswith('.ytd'):
                    texture_letter = texture_letter[:-4]
                if texture_letter is not None:
                    if item_number not in clothes_dict[root_name][category]:
                        clothes_dict[root_name][category][item_number] = {"textures": []}
                    if texture_letter not in clothes_dict[root_name][category][item_number]["textures"]:
                        clothes_dict[root_name][category][item_number]["textures"].append(texture_letter)
                        clothes_dict[root_name][category][item_number]["textures"].sort()

current_directory = os.path.dirname(os.path.realpath(__file__))
parent_directory = os.path.dirname(os.path.dirname(current_directory))
target_directory = os.path.join(parent_directory, "web", "build")
output_file_path = os.path.join(target_directory, "clothes.json")

# Ensure the target directory exists
os.makedirs(target_directory, exist_ok=True)

# Save the clothes.json file
with open(output_file_path, 'w') as output_file:
    json.dump(clothes_dict, output_file)

file_size_bytes = os.path.getsize(output_file_path)
file_size_kb = file_size_bytes / 1024
print(f"{TerminalColors.OKGREEN}Done! | Processed {total_files} files | File Size: {file_size_kb:.2f}KB{TerminalColors.ENDC}")

# HTML Documentation as a string
readme_html = """
<div align="center">
<img src="https://cdn.discordapp.com/attachments/1016069609897595011/1101229011742507040/um-soup-logo.png" width=64 height=64><br><br>
</div>

# um-soup `v1.0.0`
<sup>um-soup is a convenient tool designed to help you quickly find and organize clothing files in your FiveM server. Originally created to handle single-folder clothing structures, this tool has been <b>revised by putty</b> to now support all folders inside the stream directory, regardless of how many subfolders there are. No matter how cluttered or deeply nested your clothing folder is, um-soup can process `.ydd` and `.ytd` files and generate a structured `clothes.json` for your server.<br>
</sup>

## 🥣 Setup
### Download the **[release](https://github.com/alp1x/um-soup/releases/latest)** (not the source!) and **[Python 3.11.3](https://www.python.org/downloads/release/python-3113/)**

* Open the `convertJSON` folder and launch `start.bat`.
* Copy the path of your clothes stream folder (e.g., `fivem/stream`).

**Note**: Thanks to the revision by putty, the tool now supports categorized clothes folders with multiple subfolders (e.g., `stream/[male]/teef/`). You no longer need to worry about having different folders inside your stream folder.

### Right-click on the terminal to paste the path when prompted.

## 🥟 Usage
* Ensure the `um-soup` resource is added to your server:
