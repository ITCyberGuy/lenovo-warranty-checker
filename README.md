# lenovo-warranty-checker
"Python script for retrieving warranty information for Lenovo devices using their support API. The script allows querying by serial number and displays warranty status, start, and end dates

import os
import requests
import sys

def get_warranty_info(serial_numbers):
    url = 'https://supportapi.lenovo.com/v2.5/warranty'
    token = os.getenv('LENOVO_API_TOKEN')
    if not token:
        print("API token not found. Please set LENOVO_API_TOKEN environment variable.")
        sys.exit(1)

    headers = {
        'ClientID': token,
        'Content-Type': 'application/json'
    }

    try:
        if isinstance(serial_numbers, list) and len(serial_numbers) > 1:
            # For multiple serial numbers, use POST request
            data = {'Serial': ','.join(serial_numbers)}
            response = requests.post(url, headers=headers, json=data, timeout=10)
        else:
            # For a single serial number, use GET request with query parameter
            serial = serial_numbers[0] if isinstance(serial_numbers, list) else serial_numbers
            params = {'Serial': serial}
            response = requests.get(url, headers=headers, params=params, timeout=10)

        # Handle response
        if response.status_code == 200:
            data = response.json()
            # Process and display the warranty status
            if isinstance(data, list):
                for warranty_info in data:
                    display_warranty_status(warranty_info)
            else:
                display_warranty_status(data)
        else:
            print(f"Error: {response.status_code}")

    except requests.exceptions.RequestException as e:
        # Handle any requests exceptions such as timeouts, connection errors, etc.
        print(f"An error occurred while making the request: {e}")

def display_warranty_status(data):
    if 'Error' in data:
        error_code = data['Error'].get('Code', 'Unknown')
        error_message = data['Error'].get('Message', 'No error message provided.')
        print(f"Error Code: {error_code}")
        print(f"Error Message: {error_message}")
        return

    serial = data.get('Serial', 'N/A')
    in_warranty = data.get('InWarranty', 'N/A')

    # Extract the earliest start date and latest end date from all warranties
    warranties = data.get('Warranty', [])
    if warranties:
        # Initialize start and end dates
        start_dates = []
        end_dates = []

        for warranty in warranties:
            start = warranty.get('Start')
            end = warranty.get('End')
            if start:
                start_dates.append(start)
            if end:
                end_dates.append(end)
        
        # Get the earliest start date and latest end date
        warranty_start = min(start_dates) if start_dates else 'N/A'
        warranty_end = max(end_dates) if end_dates else 'N/A'
    else:
        warranty_start = 'N/A'
        warranty_end = 'N/A'

    print(f"Serial Number: {serial}")
    print(f"In Warranty: {in_warranty}")
    print(f"Warranty Start Date: {warranty_start}")
    print(f"Warranty End Date: {warranty_end}")
    print("-" * 50)

if __name__ == '__main__':
    if len(sys.argv) > 1:
        # Serial numbers provided as command-line arguments
        serial_numbers = sys.argv[1:]
    else:
        # Prompt the user to enter serial numbers
        serial_numbers_input = input("Enter serial number(s) separated by commas: ").strip()
        serial_numbers = [sn.strip() for sn in serial_numbers_input.split(',')]

    get_warranty_info(serial_numbers)
