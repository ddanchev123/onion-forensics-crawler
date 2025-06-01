# Advanced Onion Crawler

![Python](https://img.shields.io/badge/python-3.7+-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green)
![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)

This is an advanced web crawler specifically designed for crawling `.onion` sites on the Tor network. It features modular content hashing, including text, image, structural, and UI element hashing, and stores the crawled data in a MySQL database.  It is designed to be robust, handling common crawling issues such as connection errors, duplicate content, and JavaScript rendering.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Database Schema](#database-schema)
- [Excluding URLs](#excluding-urls)
- [Logging](#logging)
- [Content Hashing Modules](#content-hashing-modules)
- [Error Handling](#error-handling)
- [Contributing](#contributing)
- [License](#license)

## Features

*   **`.onion` Focused:** Specifically designed and tested for crawling `.onion` sites.
*   **Tor Network Integration:**  Uses a SOCKS5 proxy to route traffic through the Tor network.
*   **Configurable Crawl Depth:**  Control how deep the crawler explores the web of links.
*   **JavaScript Rendering:**  Optionally render JavaScript-heavy pages using `requests-html` and Chromium.
*   **Content Hashing:**  Includes MD5, SHA1, and SHA256 hashing for content identification.
*   **Image Hashing:**  Calculates perceptual hashes (pHash, dHash) and color histograms for images.
*   **Text Hashing:**  Generates sentence and document embeddings using Sentence Transformers.
*   **Structural Hashing:**  Hashes the DOM tree, CSS styles, and JavaScript functionality.
*   **UI Element Hashing:**  Hashes key UI elements, button text, and form structures.
*   **Database Storage:** Stores crawled data, including content hashes, in a MySQL database.
*   **Duplicate Detection:**  Detects and handles duplicate content using URL and content hashes.
*   **Error Logging:**  Comprehensive logging of errors and warnings to a file and console.
*   **User-Agent Rotation:**  Rotates user agents to avoid being blocked.
*   **Exclusion List:**  Allows excluding specific URLs from being crawled.
*   **Progress Bar:**  Uses `tqdm` to display a progress bar during crawling.
*   **Retry Mechanism:** Implements a retry mechanism with exponential backoff for handling request failures.
*   **Modular Design:** Uses separate modules for image, text, structural, and UI element hashing, making the code more organized and maintainable.

## Installation

1.  **Install Python 3.7+:** Ensure you have Python 3.7 or a later version installed.

2.  **Clone the Repository:**

    ```bash
    git clone <repository_url>
    cd <repository_directory>
    ```

3.  **Install Dependencies:**

    ```bash
    pip install -r requirements.txt
    ```

    Create a `requirements.txt` file with the following content:

    ```
    requests
    beautifulsoup4
    requests_html
    tqdm
    urllib3
    mysql-connector-python
    sentence-transformers
    nltk
    Pillow
    imagehash
    ```

4.  **Install NLTK Resources:**

    ```python
    import nltk
    nltk.download('punkt')
    ```

5.  **Install Tor:**  You need to have Tor installed and running on your system.  The crawler is configured to use the default Tor SOCKS5 proxy at `127.0.0.1:9150`.

    *   **Linux (Debian/Ubuntu):** `sudo apt-get install tor`
    *   **macOS:** `brew install tor` (using Homebrew)
    *   **Windows:** Download the Tor Browser Bundle from the official Tor Project website and run the Tor service.

6.  **Set up MySQL Database:**

    *   Install MySQL Server.
    *   Create a database named `onion_crawler` (or whatever you configure in `db_name`).
    *   Create a user with appropriate privileges for the database (or use your root user, but it's not recommended for security reasons).

## Configuration

1.  **Database Configuration:**  Edit the `onion_forensics_crawler.py` file to set your MySQL database credentials:

    ```python
    db_host = "localhost"
    db_name = "onion_crawler_01"
    db_user = "onion_crawler_01"
    db_password = "onion_crawler_01"
    ```

2.  **Tor Proxy:**  The crawler assumes Tor is running on `127.0.0.1:9150`.  If your Tor proxy is configured differently, update the `proxies` dictionary in the `AdvancedWebCrawler` class:

    ```python
    self.proxy_ip = "127.0.0.1"
    self.proxy_port = 9150
    self.proxies = {
        'http': f'socks5h://{self.proxy_ip}:{self.proxy_port}',
        'https': f'socks5h://{self.proxy_ip}:{self.proxy_port}'
    }
    ```

3.  **Chromium Path (Optional):** If you want to use JavaScript rendering and have a specific Chromium executable, set the `chromium_path` variable:

    ```python
    chromium_path = "/path/to/chromium"  # Example: /usr/bin/chromium
    ```

    If `chromium_path` is not set, `requests-html` will attempt to download Chromium automatically.

4.  **Exclusion File:** Create a file named `exclude.txt` (or whatever you configure in `exclude_file`) in the same directory as the script.  Add URLs to this file, one per line, that you want to exclude from crawling.  Only `.onion` URLs are allowed.

5.  **Input File:** Create a file named `input.txt` (or whatever you specify when running the script) in the same directory as the script. Add the `.onion` URLs you want to start crawling from, one per line.

## Usage

1.  **Run the Crawler:**

    ```bash
    python onion_forensics_crawler.py
    ```

2.  **Command-Line Arguments:**  You can modify the crawler's behavior by editing the `if __name__ == '__main__':` section of the script.  For example, to change the crawl depth:

    ```python
    if __name__ == '__main__':
        output_dir = "crawled_data"
        max_depth = 2  # Default crawl depth
        render_js = False  # Disable JavaScript rendering
        output_file_prefix = "output"
        chromium_path = None
        max_retries = 3  # Maximum number of retries for requests  # Changed variable name here
        request_timeout = 60  # Increased timeout to 60 seconds
        db_host = "localhost"  # MySQL database host
        db_name = "onion_crawler_01"  # MySQL database name
        db_user = "onion_crawler_01"
        db_password = "onion_crawler_01"
        exclude_file = "exclude.txt"

        try:
            crawler = AdvancedWebCrawler(output_dir=output_dir, max_depth=max_depth, render_js=render_js,
                                             output_file_prefix=output_file_prefix,
                                             max_retries=max_retries, request_timeout=request_timeout, db_host=db_host,
                                             db_name=db_name, db_user=db_user, db_password=db_password, chromium_path=chromium_path, exclude_file=exclude_file)
            crawler.run()

        except Exception as e:
            logger.critical(f"A critical error occurred during initialization: {e}")
            logger.critical(traceback.format_exc())
    ```

    You can also specify a different input file:

    ```python
    crawler.run(input_file="my_onion_urls.txt")
    ```

    And change the crawl depth:

    ```python
    crawler.run(input_file="my_onion_urls.txt", crawl_depth=5)
    ```

## Database Schema

The crawler stores data in a MySQL database table named `crawled_data`.  Here's the recommended schema:

```sql
CREATE TABLE crawled_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    url_hash VARCHAR(32) NOT NULL UNIQUE,
    is_duplicate BOOLEAN DEFAULT FALSE,
    url TEXT NOT NULL,
    md5 VARCHAR(32),
    sha1 VARCHAR(40),
    sha256 VARCHAR(64),
    scripts JSON,
    initial_depth INT,
    image_md5 VARCHAR(32),
    image_sha1 VARCHAR(32),
    image_sha256 VARCHAR(32),
    image_phash VARCHAR(32),
    image_dhash VARCHAR(32),
    image_color_histogram JSON,
    sentence_embeddings JSON,
    document_embeddings JSON,
    dom_tree_md5 VARCHAR(32),
    dom_tree_sha1 VARCHAR(32),
    dom_tree_sha256 VARCHAR(32),
    css_style_md5 VARCHAR(32),
    css_style_sha1 VARCHAR(32),
    css_style_sha256 VARCHAR(32),
    js_functionality_md5 VARCHAR(32),
    js_functionality_sha1 VARCHAR(32),
    js_functionality_sha256 VARCHAR(32),
    key_elements_md5 VARCHAR(32),
    key_elements_sha1 VARCHAR(32),
    key_elements_sha256 VARCHAR(32),
    button_text_md5 VARCHAR(32),
    button_text_sha1 VARCHAR(32),
    button_text_sha256 VARCHAR(32),
    form_structure_md5 VARCHAR(32),
    form_structure_sha1 VARCHAR(32),
    form_structure_sha256 VARCHAR(32),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
