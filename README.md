#!/bin/bash

# Set API keys
HIBP_API_KEY="your_api_key"  # Have I Been Pwned API
NUMVERIFY_API_KEY="your_api_key"  # NumVerify API
HUNTER_API_KEY="your_api_key"  # Hunter.io API
PLATE_API_KEY="your_api_key"  # Vehicle registry API

# Function to check email breaches
check_email_breaches() {
    email="$1"
    url="https://haveibeenpwned.com/api/v3/breachedaccount/$email"
    response=$(curl -s -H "hibp-api-key: $HIBP_API_KEY" "$url")
    
    if [[ "$response" == *"breaches"* ]]; then
        echo "Breaches found for $email: $response"
    else
        echo "No breaches found for $email"
    fi
}

# Function to perform WHOIS lookup
whois_lookup() {
    domain="$1"
    echo "WHOIS information for $domain:"
    whois "$domain" | grep -E "Registrar|Registrant|Creation Date|Updated Date|Expiration Date"
}

# Function to perform phone number lookup
phone_lookup() {
    phone="$1"
    url="http://apilayer.net/api/validate?access_key=$NUMVERIFY_API_KEY&number=$phone&format=1"
    response=$(curl -s "$url" | jq '.')
    
    if [[ "$response" == *"valid\":true"* ]]; then
        echo "Phone lookup result: $response"
    else
        echo "Phone lookup failed."
    fi
}

# Function to find social media accounts linked to an email
email_social_lookup() {
    email="$1"
    domain="${email#*@}"
    url="https://api.hunter.io/v2/email-finder?domain=$domain&email=$email&api_key=$HUNTER_API_KEY"
    response=$(curl -s "$url" | jq '.')
    
    if [[ "$response" == *"data"* ]]; then
        echo "Social media lookup for $email: $response"
    else
        echo "No social profiles found for $email."
    fi
}

# Function to perform license plate lookup
plate_lookup() {
    plate="$1"
    url="https://api.vehicleregistry.com/lookup?plate=$plate&api_key=$PLATE_API_KEY"
    response=$(curl -s "$url" | jq '.')
    
    if [[ "$response" == *"vehicle"* ]]; then
        echo "License plate lookup result: $response"
    else
        echo "License plate lookup failed."
    fi
}

# Function to generate Google search URL
google_search() {
    query="$1"
    search_url="https://www.google.com/search?q=${query// /+}"
    echo "Google Search URL: $search_url"
}

# Function to generate Ahmia (deep web) search URL
deep_web_search() {
    query="$1"
    search_url="https://ahmia.fi/search/?q=${query// /+}"
    echo "Deep Web Search URL: $search_url"
}

# Prompt user for input
echo "Enter an email:"
read email
echo "Enter a phone number (with country code):"
read phone
echo "Enter a license plate:"
read plate

# Run functions
check_email_breaches "$email"
whois_lookup "${email#*@}"
phone_lookup "$phone"
email_social_lookup "$email"
plate_lookup "$plate"
google_search "$email"
deep_web_search "$email"
