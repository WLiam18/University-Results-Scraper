# University-Results-Scraper
import csv
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service as ChromeService
from selenium.webdriver.chrome.options import Options
from bs4 import BeautifulSoup
from google.colab import files  # Import the files module

# Set the path to ChromeDriver
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')
chrome_options.binary_location = '/usr/bin/chromium-browser'

# Function to scrape results and save as a CSV file
def scrape_and_save_to_csv(reg_no, dob, output_file):
    # Define the URL of the results page
    url = 'https://egovernance.unom.ac.in/results/ugresult.asp'

    # Initialize ChromeDriver without the executable_path argument
    driver = webdriver.Chrome(options=chrome_options)

    try:
        # Open the website
        driver.get(url)

        # Find the registration number input field and enter the registration number
        regno_input = driver.find_element(By.NAME, 'regno')
        regno_input.send_keys(reg_no)

        # Find the date of birth input field and enter the date of birth
        dob_input = driver.find_element(By.NAME, 'pwd')
        dob_input.send_keys(dob)

        # Find the "Get Result" button and click it
        get_result_button = driver.find_element(By.XPATH, "//input[@value='Get Result']")
        get_result_button.click()

        # Wait for the results page to load
        driver.implicitly_wait(10)

        # Get the entire page content and parse it with BeautifulSoup
        page_content = driver.page_source
        soup = BeautifulSoup(page_content, 'html.parser')

        # Find and extract the specific tables within tbody elements
        table4 = soup.find_all('tbody')[3]

        # Extract individual results from table4_data and format them
        results = table4.find_all('tr')[1:]  # Skip the header row
        formatted_results = []

        for result in results:
            cols = result.find_all('td')
            formatted_row = [col.text.strip() for col in cols]
            formatted_results.append(formatted_row)

        # Save the formatted results as a CSV file
        with open(output_file, 'w', newline='') as csv_file:
            writer = csv.writer(csv_file)
            writer.writerows(formatted_results)

        # Prompt download of the CSV file
        files.download(output_file)  # Download the file

    finally:
        # Close the web browser when done
        driver.quit()

# Example usage
if __name__ == "__main__":
    # Replace these with actual student registration numbers and dates of birth
    student_data = [
        {"reg_no": "212103403", "dob": "18/09/2003"},
        {"reg_no": "212103402", "dob": "04/10/2003"}
    ]

    for student in student_data:
        output_file = f"{student['reg_no']}_results.csv"
        scrape_and_save_to_csv(student["reg_no"], student["dob"], output_file)

This project is a web scraper designed to extract and format university examination results from a specific website. It automates the process of retrieving results, cleans up the data, and presents it in a structured format.

