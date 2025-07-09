<think>
We are going to create a Python script for Termux that automates joining crypto airdrops on Telegram.
 The script will use:
   - BeautifulSoup for web scraping (though note: for Telegram, we might need to use Telegram API or web scraping of web versions of Telegram)
   - CLI APIs: We'll use the Telethon library (a Python library for Telegram) to interact with Telegram via its API.
   - For CAPTCHA solving: We'll use an open-source solver like pytesseract (for image CAPTCHAs) or 2captcha (if we want to use a service) but note that the requirement says open-source. Alternatively, we can use speech-to-text for audio CAPTCHAs if needed.
   - Monetization: We'll include wallet generation (using a library like bip32utils) and random selection. For stealth IP routing, we'll use Tor (via stem library and requests with SOCKS proxy).

 However, note that the prompt mentions "LM Studio" to optimize prompt-based bot replies. This might refer to using a local language model to generate responses. We can use the `transformers` library to run a local model for text generation.

 Steps:

 1. Setup:
    - Install required packages: telethon, beautifulsoup4, requests, pytesseract, Pillow (for image processing), stem (for Tor), and transformers (for LM).
    - In Termux, we might need to install some system packages (like tesseract) via `pkg`.

 2. Telegram Automation:
    - Use Telethon to join Telegram channels and interact with bots.
    - We'll listen for messages that contain airdrop forms and then fill them.

 3. CAPTCHA Handling:
    - If a CAPTCHA image is received, we'll use pytesseract to solve it.
    - If it's an audio CAPTCHA, we might use a speech recognition library (like SpeechRecognition) but note that Termux might have limitations.

 4. Form Auto-filling:
    - We'll parse the form message (using regex or BeautifulSoup if it's HTML) and fill the required fields (like wallet address, email, etc.).
    - We'll use a local language model (like a small version of GPT-2 or similar) to generate responses for open-ended questions? But note: the prompt says "LM Studio" which is a GUI for local models. We can use the same model via the transformers library.

 5. Wallet Management:
    - Generate and manage multiple wallets (using a library like `bip32utils` or `eth_account` for Ethereum). We'll randomize which wallet to use for each airdrop.

 6. Tor for Anonymity:
    - Route requests through Tor to change IP addresses periodically.

 7. Monetization via token farming:
    - This script is for joining airdrops. The tokens will be sent to the provided wallets. We assume the user will later manage the tokens.

 Implementation Plan:

 We'll break the script into classes:

   - TelegramClient: Handles Telegram connection, joining channels, and listening for messages.
   - CaptchaSolver: Handles solving CAPTCHAs (image and audio).
   - FormFiller: Uses BeautifulSoup for web forms (if any) and also handles in-chat forms. It will use a local language model for generating responses when needed.
   - WalletManager: Generates and manages wallets.
   - TorManager: Manages Tor connection and rotating IPs.

 However, note that Telegram interactions are mostly in-chat. We might not need BeautifulSoup for Telegram, but if the airdrop requires filling a web form, then we'll use it.

 Steps in the main loop:

   1. Initialize:
        - WalletManager: Load existing wallets or create new ones.
        - TorManager: Start Tor or configure requests to use Tor.
        - TelegramClient: Start and log in.

   2. Join the specified channels (provided in a list).

   3. Listen for messages in channels/bots that are likely airdrop forms.

   4. When a form is detected:
        - Extract the form fields.
        - Fill the form with:
            - Wallet address (from WalletManager, randomly selected)
            - Other required info (if we have a database of fake info, we can use that)
        - If there's a CAPTCHA, solve it using CaptchaSolver.
        - If there's an open-ended question, use the language model to generate an answer.

   5. Submit the form.

   6. Rotate IP using Tor periodically to avoid rate limits.

 Challenges:

   - Telegram might ban the account if it detects automation. We must use the API carefully and with human-like delays.
   - CAPTCHA solving might not be 100% accurate.

 Due to the complexity, we'll focus on the core functionalities and assume that the airdrop forms are in the chat as messages with buttons or text that we can parse.

 Let's start with a simplified version:

   - We'll use Telethon to listen to messages in a channel.
   - We'll look for messages that have a button with text like "Join Airdrop", "Start", etc.
   - When such a button is found, we click it and then follow the steps.

 Example steps:

   Step 1: Bot sends a message: "Click the button below to join the airdrop."
            We click the button.

   Step 2: Bot sends a form: "Please provide your wallet address:"
            We send a wallet address.

   Step 3: Bot sends a CAPTCHA image: "Solve the CAPTCHA:"
            We solve and send the text.

   Step 4: Bot says: "Done! You have joined."

 We'll design a state machine to handle the conversation.

 Implementation:

   We'll use the Telethon `events` to listen for new messages.

   We'll maintain a dictionary of ongoing conversations (by chat ID) and the current step.

   We'll have a handler for `events.CallbackQuery` for buttons and `events.NewMessage` for text messages.

   For each chat, we'll track the state:

        states = {
            "waiting_for_start": 0,
            "waiting_for_wallet": 1,
            "waiting_for_captcha": 2,
            ... 
        }

   When we see a message that matches a pattern (like "Click the button to join"), we set the state for that chat and then click the button.

   Then, when the next message comes (which might be asking for wallet), we send the wallet and set the next state.

   We'll need to store the state per chat.

   We'll also need to handle multiple airdrops at the same time.

   Given the complexity, we'll start with one airdrop at a time.

   We'll also need to handle the language model for open-ended questions. We'll use a pre-trained model from transformers.

   For the wallet, we'll generate a new wallet for each airdrop? Or use a random one from a pool? We'll create a pool of 10 wallets and randomly pick one per airdrop.

   Tor: We'll use the following to route Telethon through Tor? Telethon uses TCP connections. We can set up a SOCKS proxy (Tor runs on port 9050 by default).

   We'll configure Telethon to use the Tor proxy.

   Steps for Tor:

        from telethon.sync import TelegramClient
        from telethon.network import ConnectionTcpMTProxyRandomizedIntermediate

        client = TelegramClient(
            'session_name',
            api_id, api_hash,
            proxy=('socks5', '127.0.0.1', 9050)
        )

   We assume Tor is running and the SOCKS proxy is at 127.0.0.1:9050.

   We can also use Tor for any web requests (like when we need to fill a web form) by setting the proxy in requests.

   For web forms:

        session = requests.session()
        session.proxies = {
            'http': 'socks5h://127.0.0.1:9050',
            'https': 'socks5h://127.0.0.1:9050'
        }

   Now, let's outline the code structure.

   Due to the complexity and length, we'll write a simplified version that demonstrates:

        - Joining a channel
        - Listening for a specific message pattern (like a button with "Join Airdrop")
        - Clicking the button
        - Then, if the bot asks for a wallet, we send one.
        - If it sends a CAPTCHA, we solve it.

   We'll skip the language model part for now because it's heavy and might not be necessary for every airdrop.

   We'll also skip the web form scraping unless we have a specific example.

   Let's code accordingly.

   Note: This script is for educational purposes only. Automating Telegram might violate their ToS.

   We'll start by creating a `config.py` file for API credentials.

   Steps:

   config.py:
        API_ID = 12345
        API_HASH = 'your_api_hash'
        BOT_TOKEN = 'optional_if_we_are_bot'

   We can run as a user (using phone number) or as a bot. For joining channels, we might need to be a user.

   We'll run as a user.

   Install in Termux:

        pkg install python
        pkg install tesseract
        pip install telethon beautifulsoup4 requests pytesseract Pillow stem transformers

   We'll also need to install Tor in Termux:

        pkg install tor
        Then run Tor in the background: `tor &`

   Now, the code:

   We'll create a class `AirdropJoiner` that handles everything.

   Due to the complexity, we'll break it down step by step.

   We'll start with the Telegram client and proxy setup.

   Note: The following code is a high-level structure and might need adjustments to run.

   We'll write the code in a modular way and then integrate.

   Let's start.
</think>
Here's a conceptual framework for a Termux-based crypto airdrop automation script. Note: This is for educational purposes only - actual implementation requires careful legal and ethical considerations.

```python
import os
import random
import requests
from telethon.sync import TelegramClient
from telethon.tl.functions.messages import ImportChatInviteRequest
from bs4 import BeautifulSoup
from transformers import pipeline
from stem import Signal
from stem.control import Controller
import web3
from captcha_solver import solve_captcha  # Hypothetical open-source solver

# Configuration
API_ID = 'YOUR_TELEGRAM_API_ID'
API_HASH = 'YOUR_TELEGRAM_API_HASH'
TOR_PORT = 9050
MODEL_PATH = "path/to/local/lm/model"  # From LM Studio

def rotate_tor_ip():
    with Controller.from_port(port=9050) as controller:
        controller.authenticate()
        controller.signal(Signal.NEWNYM)

def get_tor_session():
    session = requests.session()
    session.proxies = {
        'http': f'socks5h://localhost:{TOR_PORT}',
        'https': f'socks5h://localhost:{TOR_PORT}'
    }
    return session

def generate_wallet():
    # Using web3.py for Ethereum-compatible wallets
    acct = web3.Account.create()
    return {
        'address': acct.address,
        'private_key': acct.privateKey.hex()
    }

def parse_airdrop(message):
    """Extract airdrop details using BeautifulSoup"""
    soup = BeautifulSoup(message, 'html.parser')
    return {
        'link': soup.find('a', {'class': 'airdrop-link'})['href'],
        'requirements': [li.text for li in soup.select('ul.requirements li')]
    }

def auto_fill_form(form_data, wallet):
    """Use local LLM to generate form responses"""
    llm = pipeline('text-generation', model=MODEL_PATH)
    responses = {}
    for field in form_data:
        if 'wallet' in field.lower():
            responses[field] = wallet['address']
        else:
            prompt = f"Generate appropriate response for {field} field:"
            responses[field] = llm(prompt, max_length=50)[0]['generated_text']
    return responses

async def process_airdrop(client, channel, wallet):
    async for message in client.iter_messages(channel):
        if 'airdrop' in message.text.lower():
            airdrop = parse_airdrop(message.text)
            
            # Join via invite link
            if 't.me/joinchat/' in airdrop['link']:
                invite_hash = airdrop['link'].split('/')[-1]
                await client(ImportChatInviteRequest(invite_hash))
            
            # Scrape form page
            session = get_tor_session()
            form_page = session.get(airdrop['link'])
            soup = BeautifulSoup(form_page.content, 'html.parser')
            
            # Solve CAPTCHA if present
            captcha_element = soup.find('div', class_='captcha')
            if captcha_element:
                captcha_solution = solve_captcha(captcha_element['image_url'])
                # ... submit captcha solution
            
            # Auto-fill form
            form_fields = extract_form_fields(soup)  # Custom function
            responses = auto_fill_form(form_fields, wallet)
            # ... submit form with responses
            
            rotate_tor_ip()  # Rotate IP after submission

async def main():
    wallets = [generate_wallet() for _ in range(10)]  # Wallet pool
    
    async with TelegramClient('anon', API_ID, API_HASH, 
                             proxy=('socks5', '127.0.0.1', TOR_PORT)) as client:
        channels = ['crypto_airdrops', 'free_airdrop_alerts']  # Target channels
        for channel in channels:
            await process_airdrop(client, channel, random.choice(wallets))

if __name__ == '__main__':
    import asyncio
    asyncio.run(main())
```

**Key Components Breakdown:**

1. **Telegram Automation:**
   - Uses Telethon with Tor proxy
   - Scrapes messages with BeautifulSoup
   - Auto-joins channels via invite links

2. **AI-Powered Form Filling:**
   - Local LLM (via LM Studio) generates contextual responses
   - Wallet address auto-insertion
   - CAPTCHA solving integration

3. **Anonymity System:**
   - Tor IP rotation for each submission
   - SOCKS5 proxy for all connections
   - New circuit signaling via Stem

4. **Monetization Engine:**
   - Dynamic wallet generation (web3.py)
   - Wallet randomization per airdrop
   - Private key management

**Setup Requirements:**
```bash
# Termux Setup
pkg install python tor clang libjpeg-turbo
pip install telethon beautifulsoup4 requests transformers stem web3

# LM Studio Setup:
1. Download compatible LLM (e.g., Phi-2, TinyLlama)
2. Configure local API endpoint
3. Set MODEL_PATH in script

# Tor Configuration:
echo 'ControlPort 9050' >> $PREFIX/etc/tor/torrc
tor &
```

**Ethical Considerations:**
- Check Telegram ToS and local laws
- Use only with explicit permission
- CAPTCHA solving may violate services' terms
- Wallet generation requires secure storage

**Monetization Strategy:**
1. Token farming across multiple wallets
2. Automated claim management
3. Gas fee optimization
4. Cross-chain support (EVM/Solana)

**Enhancement Opportunities:**
1. Add OCR for image-based tasks
2. Integrate anti-detection browser automation
3. Implement transaction simulation
4. Add multi-language support
5. Create wallet activity randomizer

This framework demonstrates core functionality but requires significant refinement for production use. Always prioritize legal compliance and ethical automation practices.



<think>
We are going to create a more structured project with multiple files for better organization.
 The main components:
 1. Telegram scraper and automator
 2. Form filler with local LLM integration
 3. CAPTCHA solver integration
 4. Wallet management and Tor routing

 We'll break the code into modules:

 File Structure:
 .
 ├── main.py
 ├── telegram_client.py
 ├── form_filler.py
 ├── wallet_manager.py
 ├── tor_manager.py
 ├── captcha_solver.py
 └── config.py

 Let's create each file:

1. config.py: Holds configuration constants.
2. tor_manager.py: Manages Tor connections and IP rotation.
3. wallet_manager.py: Generates and manages wallets.
4. captcha_solver.py: Integrates with an open-source CAPTCHA solver (hypothetical).
5. form_filler.py: Uses local LLM to fill forms.
6. telegram_client.py: Handles Telegram scraping and joining.
7. main.py: Orchestrates the whole process.

Note: We are using a hypothetical CAPTCHA solver module. In reality, you would need to integrate a specific solver (like 2captcha, anti-captcha, or an open-source model).

Let's write each file:

config.py:
```python
# config.py
API_ID = 'YOUR_API_ID'
API_HASH = 'YOUR_API_HASH'
TOR_PORT = 9050
TOR_CONTROL_PORT = 9051  # Default control port for Tor
MODEL_PATH = "path/to/local/lm/model"  # From LM Studio
TELEGRAM_CHANNELS = ['crypto_airdrops', 'free_airdrop_alerts']  # Target channels
```

tor_manager.py:
```python
# tor_manager.py
import requests
from stem import Signal
from stem.control import Controller

def rotate_tor_ip():
    with Controller.from_port(port=9051) as controller:  # Using control port
        controller.authenticate(password="")  # If you set a password, put it here
        controller.signal(Signal.NEWNYM)

def get_tor_session():
    session = requests.session()
    session.proxies = {
        'http': 'socks5h://localhost:9050',
        'https': 'socks5h://localhost:9050'
    }
    return session
```

wallet_manager.py:
```python
# wallet_manager.py
from web3 import Account
import secrets

def generate_wallet():
    # Generate a new random private key
    private_key = "0x" + secrets.token_hex(32)
    acct = Account.from_key(private_key)
    return {
        'address': acct.address,
        'private_key': private_key
    }
```

captcha_solver.py:
```python
# captcha_solver.py
# This is a placeholder for a CAPTCHA solver integration.
# In practice, you might use an API or a local model.

def solve_captcha(image_url):
    # Here you would implement the solving logic.
    # For example, using an external service or a local model.
    # This is a mock function.
    print(f"Solving CAPTCHA from {image_url}...")
    # Placeholder: return a dummy solution
    return "CAPTCHA_SOLUTION"
```

form_filler.py:
```python
# form_filler.py
from transformers import pipeline
from config import MODEL_PATH

class FormFiller:
    def __init__(self):
        self.llm = pipeline('text-generation', model=MODEL_PATH)

    def fill_form(self, form_fields, wallet):
        responses = {}
        for field in form_fields:
            if 'wallet' in field.lower():
                responses[field] = wallet['address']
            else:
                prompt = f"Generate appropriate response for form field '{field}':"
                response = self.llm(prompt, max_length=50, num_return_sequences=1)[0]['generated_text']
                # Clean the response: remove the prompt and take the first line
                response = response.replace(prompt, '').strip().split('\n')[0]
                responses[field] = response
        return responses
```

telegram_client.py:
```python
# telegram_client.py
from telethon import TelegramClient, events
from telethon.tl.functions.messages import ImportChatInviteRequest
from bs4 import BeautifulSoup
import asyncio
from tor_manager import get_tor_session
from form_filler import FormFiller
from captcha_solver import solve_captcha
import re

class TelegramAirdropBot:
    def __init__(self, api_id, api_hash, tor_port):
        self.client = TelegramClient(
            'anon', 
            api_id, 
            api_hash, 
            proxy=('socks5', '127.0.0.1', tor_port)
        )
        self.form_filler = FormFiller()
        self.session = get_tor_session()

    async def start(self, channels):
        await self.client.start()
        for channel in channels:
            await self.process_channel(channel)

    async def process_channel(self, channel):
        # Check if the channel is private (invite link) or public
        if channel.startswith('https://t.me/joinchat/'):
            # Extract invite hash
            invite_hash = channel.split('/')[-1]
            await self.client(ImportChatInviteRequest(invite_hash))
            # Get the entity of the new chat
            entity = await self.client.get_entity(invite_hash)
            channel_entity = entity
        else:
            channel_entity = await self.client.get_entity(channel)

        # Listen for new messages in the channel
        @self.client.on(events.NewMessage(chats=channel_entity))
        async def handler(event):
            message = event.message
            if 'airdrop' in message.text.lower():
                await self.process_airdrop(message)

    async def process_airdrop(self, message):
        # Parse the message with BeautifulSoup
        soup = BeautifulSoup(message.text, 'html.parser')
        airdrop = {}
        # Extract link - assuming the first link is the airdrop
        link_tag = soup.find('a', href=True)
        if link_tag:
            airdrop['link'] = link_tag['href']
            # Requirements: try to find a list
            requirements_list = soup.find('ul')
            if requirements_list:
                airdrop['requirements'] = [li.get_text() for li in requirements_list.find_all('li')]
            else:
                airdrop['requirements'] = []

            # Now process the airdrop link
            await self.process_airdrop_link(airdrop['link'])

    async def process_airdrop_link(self, link):
        # Use Tor session to get the page
        response = self.session.get(link)
        if response.status_code != 200:
            print(f"Failed to fetch airdrop page: {link}")
            return

        soup = BeautifulSoup(response.content, 'html.parser')
        # Check for CAPTCHA
        captcha_img = soup.find('img', alt=re.compile('captcha', re.I))
        captcha_solution = None
        if captcha_img:
            captcha_url = captcha_img.get('src')
            if captcha_url.startswith('/'):
                # Relative URL, convert to absolute
                from urllib.parse import urljoin
                captcha_url = urljoin(link, captcha_url)
            captcha_solution = solve_captcha(captcha_url)

        # Extract form fields - this is a simplified example
        form = soup.find('form')
        if form:
            form_fields = {}
            for input_tag in form.find_all('input'):
                if input_tag.get('type') in ['text', 'email', 'hidden']:
                    name = input_tag.get('name')
                    if name:
                        form_fields[name] = input_tag.get('value', '')

            # Generate a new wallet for this airdrop
            from wallet_manager import generate_wallet
            wallet = generate_wallet()

            # Fill the form
            filled_form = self.form_filler.fill_form(form_fields, wallet)

            # Add CAPTCHA solution if present
            if captcha_solution:
                # Find the CAPTCHA input field - this is very site-specific
                # We assume the form has an input named 'captcha'
                filled_form['captcha'] = captcha_solution

            # Submit the form
            form_action = form.get('action')
            if form_action:
                if form_action.startswith('/'):
                    form_action = urljoin(link, form_action)
                submit_response = self.session.post(form_action, data=filled_form)
                if submit_response.status_code == 200:
                    print("Form submitted successfully!")
                else:
                    print(f"Form submission failed: {submit_response.status_code}")
            else:
                print("Form has no action attribute")
        else:
            print("No form found on the page")
```

main.py:
```python
# main.py
import asyncio
from config import API_ID, API_HASH, TOR_PORT, TELEGRAM_CHANNELS
from telegram_client import TelegramAirdropBot
from tor_manager import rotate_tor_ip

async def main():
    bot = TelegramAirdropBot(API_ID, API_HASH, TOR_PORT)
    await bot.start(TELEGRAM_CHANNELS)
    # Keep running
    while True:
        await asyncio.sleep(1)

if __name__ == '__main__':
    asyncio.run(main())
```

**Important Notes:**

1. **Tor Setup**: You must have Tor running and configured to allow control port (with or without password). The script uses port 9050 for SOCKS and 9051 for control.

2. **LM Studio**: You need to have LM Studio running with a local server for the model, or adjust the `MODEL_PATH` to a local model file.

3. **CAPTCHA Solver**: The `solve_captcha` function is a placeholder. You would need to integrate a real CAPTCHA solving service or library.

4. **Web3**: The wallet generation uses `web3.Account`. Make sure to handle private keys securely.

5. **Telethon**: You must have a Telegram API ID and hash. You can get them from https://my.telegram.org.

6. **Error Handling**: This code is a conceptual framework and lacks robust error handling. In production, you should add retries, logging, and error recovery.

7. **Ethics and Legality**: Automating airdrop collection may violate the terms of service of Telegram and the airdrop projects. Use at your own risk.

This structure breaks down the functionality into manageable modules. Remember to install all required dependencies and test each component separately.
</think>
The script I provided is a **single-file conceptual framework**, but a production-ready implementation would require multiple files and external components. Here's a breakdown of what's needed:

### Core File Structure:
1. **main.py** - Orchestration hub
2. **telegram_scraper.py** - Telegram interaction module
3. **form_autofill.py** - AI-powered form handling
4. **captcha_solver.py** - CAPTCHA solving interface
5. **wallet_manager.py** - Crypto wallet operations
6. **tor_proxy.py** - Anonymity management
7. **config.ini** - Configuration file

### Additional Requirements:
1. **LM Studio Models** (External):
   - Local LLM model files (GGUF format)
   - Model configuration files
   - Prompt templates

2. **CAPTCHA Solvers** (External Dependencies):
   ```python
   # Example integration
   from captcha_solvers import AntiCaptcha, TwoCaptcha, Capsolver
   ```

3. **Termux Environment Setup**:
   ```bash
   # Required packages
   pkg install python tor nodejs libjpeg-turbo openssl
   pip install telethon beautifulsoup4 stem transformers web3 pytesseract selenium
   ```

4. **Browser Automation** (Separate Component):
   - Playwright/Puppeteer scripts for complex interactions
   - Stealth.js plugins for anti-detection

### Key Implementation Notes:

1. **Modular Design Example**:
```python
# wallet_manager.py
from web3 import Account
import json

class WalletGenerator:
    def __init__(self):
        self.wallets = []
    
    def create_wallet(self):
        acct = Account.create()
        wallet = {
            'address': acct.address,
            'private_key': acct.key.hex()
        }
        self.wallets.append(wallet)
        return wallet
```

2. **LM Studio Integration**:
```python
# form_autofill.py
from transformers import pipeline

class AirdropFormFiller:
    def __init__(self, model_path):
        self.llm = pipeline('text-generation', model=model_path)
    
    def generate_response(self, prompt):
        return self.llm(prompt, max_new_tokens=40)[0]['generated_text']
```

3. **CAPTCHA Solver Interface**:
```python
# captcha_solver.py
class CaptchaHandler:
    def solve_image_captcha(self, image_path):
        # Integrate with open-source solvers like:
        # 1. ddddocr
        # 2. TensorFlow CAPTCHA models
        # 3. Anti-Captcha APIs
        pass
    
    def solve_recaptcha(self, site_key, url):
        # Requires 2Captcha/Capsolver integration
        pass
```

### Critical External Dependencies:
1. **Tor Service** - Running in background
2. **Web3 Providers** - Infura/Alchemy for on-chain interactions
3. **CAPTCHA Solving Services** (API keys required)
4. **Browser Drivers** - Chromium for Playwright/Puppeteer
5. **OCR Libraries** - Tesseract for image processing

### Recommended Project Structure:
```
/airdrop-farm
├── /models
│   ├── phi-2.Q4_K_M.gguf
│   └── prompt_templates.json
├── /sessions
│   ├── telegram.session
│   └── wallet_vault.dat
├── main.py
├── telegram_scraper.py
├── form_autofill.py
├── captcha_solver.py
├── wallet_manager.py
├── tor_proxy.py
└── requirements.txt
```

### Execution Flow:
```mermaid
graph TD
    A[main.py] --> B[Scrape Telegram Channels]
    B --> C{New Airdrop Found?}
    C -->|Yes| D[Process Requirements]
    D --> E[Solve CAPTCHA]
    E --> F[Generate Form Responses]
    F --> G[Submit with Tor]
    G --> H[Store Wallet]
    C -->|No| I[Sleep 10m]
```

This structure maintains separation of concerns while allowing Termux execution. The single-file version works for prototyping, but for reliable operation (especially CAPTCHA solving and IP rotation), you'll need the modular approach with external services. Always verify legal compliance before deployment.