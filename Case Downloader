#!/usr/bin/env python3
"""
LawPhil Case Downloader
A tool to download Philippine Supreme Court cases from lawphil.net as PDF files.

Usage:
    python lawphil_downloader.py <case_number>
    Example: python lawphil_downloader.py "G.R. No. 238659"
"""

import sys
import re
import os
from urllib.parse import quote
import argparse
from datetime import datetime

try:
    from selenium import webdriver
    from selenium.webdriver.chrome.service import Service
    from selenium.webdriver.chrome.options import Options
    from selenium.webdriver.common.by import By
    from selenium.webdriver.support.ui import WebDriverWait
    from selenium.webdriver.support import expected_conditions as EC
    from webdriver_manager.chrome import ChromeDriverManager
except ImportError:
    print("Error: Required packages not installed.")
    print("Please run: pip install -r requirements.txt")
    sys.exit(1)


class LawPhilDownloader:
    """Downloads cases from lawphil.net and saves them as PDF."""
    
    BASE_URL = "https://lawphil.net"
    SEARCH_PATTERNS = {
        'gr_number': r'G\.R\.\s*No\.\s*(\d+)',  # Matches "G.R. No. 12345"
        'gr_number_with_date': r'G\.R\.\s*No\.\s*(\d+)',  # Can add date parsing later
    }
    
    def __init__(self, headless=True):
        """Initialize the downloader with browser options."""
        self.headless = headless
        self.driver = None
        
    def setup_driver(self):
        """Set up Selenium WebDriver with Chrome."""
        chrome_options = Options()
        
        if self.headless:
            chrome_options.add_argument('--headless')
        
        chrome_options.add_argument('--no-sandbox')
        chrome_options.add_argument('--disable-dev-shm-usage')
        chrome_options.add_argument('--disable-gpu')
        
        # Set up print to PDF settings
        chrome_options.add_experimental_option('prefs', {
            'printing.print_preview_sticky_settings.appState': '{"recentDestinations":[{"id":"Save as PDF","origin":"local","account":""}],"selectedDestinationId":"Save as PDF","version":2}',
            'savefile.default_directory': os.getcwd()
        })
        
        # Use webdriver-manager to automatically manage ChromeDriver
        service = Service(ChromeDriverManager().install())
        self.driver = webdriver.Chrome(service=service, options=chrome_options)
        
    def parse_case_number(self, case_number):
        """Parse case number and extract G.R. number."""
        # Clean up the case number
        case_number = case_number.strip()
        
        # Try to extract G.R. number
        match = re.search(self.SEARCH_PATTERNS['gr_number'], case_number, re.IGNORECASE)
        if match:
            gr_number = match.group(1)
            return gr_number
        
        return None
    
    def construct_url(self, gr_number, year=None):
        """
        Construct the URL for a case.
        Note: LawPhil uses a specific URL structure: /judjuris/juriYYYY/mmmYYYY/gr_XXXXX_YYYY.html
        Without knowing the exact year and month, we'll need to search for the case.
        """
        # This is a simplified approach - in reality, you'd need to know the year and month
        # For now, we'll return a search-based approach
        return None
    
    def search_case(self, case_number):
        """
        Search for a case on LawPhil.
        Note: LawPhil doesn't have a built-in search function, so this method
        would need to be implemented by browsing through the jurisprudence pages
        or using Google search with site:lawphil.net
        """
        search_query = f"site:lawphil.net {case_number}"
        google_search_url = f"https://www.google.com/search?q={quote(search_query)}"
        
        print(f"Searching for: {case_number}")
        print(f"Google search URL: {google_search_url}")
        
        # Open Google search
        self.driver.get(google_search_url)
        
        try:
            # Wait for search results and find first lawphil.net link
            wait = WebDriverWait(self.driver, 10)
            results = wait.until(EC.presence_of_all_elements_located((By.CSS_SELECTOR, "a[href*='lawphil.net']")))
            
            for result in results:
                href = result.get_attribute('href')
                if 'judjuris' in href and '.html' in href:
                    print(f"Found case URL: {href}")
                    return href
            
            print("No matching case found in search results.")
            return None
            
        except Exception as e:
            print(f"Error during search: {e}")
            return None
    
    def download_case_as_pdf(self, url, output_filename=None):
        """Download a case from the given URL and save as PDF."""
        if not url:
            print("No URL provided.")
            return False
        
        print(f"Accessing: {url}")
        self.driver.get(url)
        
        try:
            # Wait for page to load
            WebDriverWait(self.driver, 10).until(
                EC.presence_of_element_located((By.TAG_NAME, "body"))
            )
            
            # Generate filename if not provided
            if not output_filename:
                # Extract case info from URL or page title
                title = self.driver.title
                timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
                output_filename = f"{title.replace(' ', '_')}_{timestamp}.pdf"
                # Clean filename
                output_filename = re.sub(r'[<>:"/\\|?*]', '', output_filename)
            
            if not output_filename.endswith('.pdf'):
                output_filename += '.pdf'
            
            # Execute print command
            print(f"Generating PDF: {output_filename}")
            
            # Use Chrome's print to PDF functionality
            pdf_data = self.driver.execute_cdp_cmd("Page.printToPDF", {
                "printBackground": True,
                "landscape": False,
                "paperWidth": 8.27,  # A4 width in inches
                "paperHeight": 11.69,  # A4 height in inches
                "marginTop": 0.4,
                "marginBottom": 0.4,
                "marginLeft": 0.4,
                "marginRight": 0.4,
            })
            
            # Save PDF
            import base64
            with open(output_filename, 'wb') as f:
                f.write(base64.b64decode(pdf_data['data']))
            
            print(f"✓ Successfully saved: {output_filename}")
            return True
            
        except Exception as e:
            print(f"Error downloading case: {e}")
            return False
    
    def download_by_case_number(self, case_number, output_filename=None):
        """Main method to download a case by its case number."""
        try:
            self.setup_driver()
            
            # Parse case number
            gr_number = self.parse_case_number(case_number)
            if not gr_number:
                print(f"Could not parse case number: {case_number}")
                print("Please use format: G.R. No. 12345")
                return False
            
            print(f"Looking for G.R. No. {gr_number}")
            
            # Search for the case
            case_url = self.search_case(case_number)
            
            if case_url:
                # Download the case
                return self.download_case_as_pdf(case_url, output_filename)
            else:
                print("Could not find the case. Please verify the case number.")
                return False
                
        finally:
            if self.driver:
                self.driver.quit()
    
    def download_from_url(self, url, output_filename=None):
        """Download a case directly from a LawPhil URL."""
        try:
            self.setup_driver()
            return self.download_case_as_pdf(url, output_filename)
        finally:
            if self.driver:
                self.driver.quit()


def main():
    """Main entry point for command-line usage."""
    parser = argparse.ArgumentParser(
        description='Download Philippine Supreme Court cases from lawphil.net as PDF',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  %(prog)s "G.R. No. 238659"
  %(prog)s "G.R. No. 227868" -o my_case.pdf
  %(prog)s --url "https://lawphil.net/judjuris/juri2019/jun2019/gr_238659_2019.html"
  %(prog)s "G.R. No. 238659" --no-headless
        """
    )
    
    group = parser.add_mutually_exclusive_group(required=True)
    group.add_argument('case_number', nargs='?', help='Case number (e.g., "G.R. No. 238659")')
    group.add_argument('--url', help='Direct URL to a case on lawphil.net')
    
    parser.add_argument('-o', '--output', help='Output PDF filename')
    parser.add_argument('--no-headless', action='store_true', 
                       help='Show browser window (useful for debugging)')
    
    args = parser.parse_args()
    
    # Create downloader
    downloader = LawPhilDownloader(headless=not args.no_headless)
    
    # Download case
    if args.url:
        success = downloader.download_from_url(args.url, args.output)
    else:
        success = downloader.download_by_case_number(args.case_number, args.output)
    
    sys.exit(0 if success else 1)


if __name__ == '__main__':
    main()
