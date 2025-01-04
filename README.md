# Chrome-Extension-for-Amazon-Images-on-Shiprocket-Portal
Chrome extension that simplifies our order processing workflow by loading product images from Amazon onto the Shiprocket portal using the order ID. Key requirements include:

1. Automatically displaying Amazon product images on Shiprocket, with hover-zoom functionality.

2. A refresh button to reload the first product image.

3. Clicking the image should open the product page on Amazon.

4. The extension should be installable on multiple systems and work seamlessly for order verification and label printing (label script already in use).

Testing access will be provided for a smaller Amazon account, and the extension must work across different systems.
---------
To develop a Chrome extension that simplifies the order processing workflow by displaying Amazon product images on the Shiprocket portal, we will need to implement the following features:

    Fetch Amazon product images based on order ID.
    Display product images on Shiprocket with hover-zoom functionality.
    Add a refresh button to reload the first product image.
    Link the product image to the Amazon product page.
    Ensure compatibility across multiple systems for smooth operation.

Steps:

    Manifest File (manifest.json): This defines the extension's configuration, permissions, and background scripts.
    Content Script: This script will interact with the Shiprocket portal and fetch Amazon product images using order IDs.
    Popup UI (Optional): This will allow users to configure or manage the extension.
    Background Script: (Optional) If needed for additional background operations.

1. Manifest File (manifest.json)

This is the basic configuration file that defines the extension's permissions, scripts, and other details.

{
  "manifest_version": 3,
  "name": "Shiprocket Amazon Image Fetcher",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage",
    "https://www.amazon.in/*",
    "https://www.shiprocket.in/*"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": [
        "https://www.shiprocket.in/*"
      ],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "images/icon16.png",
      "48": "images/icon48.png",
      "128": "images/icon128.png"
    }
  }
}

2. Background Script (background.js)

We will use the background script to fetch Amazon product details. It will handle the requests for product image URLs based on the Amazon order ID.

// background.js

chrome.runtime.onInstalled.addListener(() => {
  console.log("Shiprocket Amazon Image Fetcher Extension Installed.");
});

async function fetchAmazonImage(orderId) {
  try {
    // Construct the Amazon URL from the order ID (this may need modification based on the Amazon link format)
    const amazonUrl = `https://www.amazon.in/dp/${orderId}`;
    const response = await fetch(amazonUrl);
    const html = await response.text();

    // Extract image URL from the HTML (assume it’s in a meta tag for simplicity)
    const imageUrl = html.match(/"mainImage":"(https:\/\/.*?)"/);
    return imageUrl ? imageUrl[1] : null;
  } catch (error) {
    console.error("Error fetching Amazon image: ", error);
    return null;
  }
}

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === "fetchAmazonImage") {
    fetchAmazonImage(message.orderId).then((imageUrl) => {
      sendResponse({ imageUrl: imageUrl });
    });
    return true; // Keep the message channel open for async response
  }
});

3. Content Script (content.js)

The content script will inject functionality into the Shiprocket portal. It will detect the order ID, fetch the Amazon product image, and display it on the Shiprocket page.

// content.js

function fetchProductImageFromOrderId(orderId) {
  chrome.runtime.sendMessage(
    { type: "fetchAmazonImage", orderId: orderId },
    (response) => {
      if (response.imageUrl) {
        displayProductImage(response.imageUrl);
      }
    }
  );
}

function displayProductImage(imageUrl) {
  const productImageContainer = document.querySelector(".product-image-container"); // Assume this container exists
  if (!productImageContainer) return;

  // Create an image element with hover-zoom functionality
  const imgElement = document.createElement("img");
  imgElement.src = imageUrl;
  imgElement.alt = "Amazon Product Image";
  imgElement.style.maxWidth = "100%";
  imgElement.style.cursor = "pointer";
  imgElement.title = "Click to view on Amazon";
  imgElement.classList.add("product-image");

  // Adding hover-zoom functionality
  imgElement.addEventListener("mouseenter", function () {
    imgElement.style.transform = "scale(1.2)";
    imgElement.style.transition = "transform 0.3s";
  });

  imgElement.addEventListener("mouseleave", function () {
    imgElement.style.transform = "scale(1)";
  });

  imgElement.addEventListener("click", function () {
    window.open(`https://www.amazon.in/dp/${orderId}`, "_blank");
  });

  // Append the image to the container
  productImageContainer.appendChild(imgElement);
}

// Look for an Order ID on Shiprocket's page (e.g., Order ID can be in the page URL or in an element)
const orderIdElement = document.querySelector(".order-id-element"); // Change this based on your specific HTML structure
if (orderIdElement) {
  const orderId = orderIdElement.textContent.trim();
  if (orderId) {
    fetchProductImageFromOrderId(orderId);
  }
}

// Add refresh button to reload the first product image
const refreshButton = document.createElement("button");
refreshButton.textContent = "Refresh Image";
refreshButton.style.position = "fixed";
refreshButton.style.top = "10px";
refreshButton.style.right = "10px";
refreshButton.style.zIndex = 1000;
refreshButton.style.padding = "10px";
refreshButton.style.backgroundColor = "#4CAF50";
refreshButton.style.color = "white";

refreshButton.addEventListener("click", () => {
  const firstProductImage = document.querySelector(".product-image");
  if (firstProductImage) {
    firstProductImage.src = "";  // Clear the existing image
    fetchProductImageFromOrderId(orderId);  // Reload the image
  }
});

document.body.appendChild(refreshButton);

4. Popup HTML (popup.html)

A simple popup UI where the user can interact with the extension, though this feature is optional and may not be needed in this case unless further configuration is required.

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Amazon Image Fetcher</title>
  </head>
  <body>
    <h1>Shiprocket Image Fetcher</h1>
    <p>Automatically display Amazon product images on Shiprocket</p>
    <button id="refresh">Refresh Product Image</button>
    <script>
      document.getElementById("refresh").addEventListener("click", () => {
        chrome.runtime.sendMessage({ type: "refreshImage" });
      });
    </script>
  </body>
</html>

5. Testing and Deployment

    Test the Extension:
        Go to chrome://extensions/ and enable Developer Mode.
        Click on Load unpacked and select the folder containing the extension.
        Visit the Shiprocket page and test if the Amazon product image loads correctly and if clicking the image opens the Amazon product page.
        Test the hover-zoom functionality and refresh button.

    Cross-System Compatibility:
        Ensure the extension works across multiple systems by testing it in different environments (such as different machines with Chrome installed).
        Make sure it doesn’t break on different versions of Shiprocket and Amazon.

Conclusion

This extension will work by leveraging the Chrome extension APIs, content scripts, and background logic to fetch Amazon product images based on the order ID and display them on the Shiprocket portal. It includes hover-zoom functionality, a refresh button, and a link to Amazon. The extension can be installed on multiple systems for streamlined order processing.

Make sure to update any CSS selectors and URLs according to the actual structure of Shiprocket and Amazon for your specific use case.
