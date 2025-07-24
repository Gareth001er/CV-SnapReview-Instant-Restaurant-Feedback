# SnapReview: Instant Restaurant Feedback

This is a google firebase studio  Next.js web app that allows users to upload a photo of a restaurant storefront, uses AI to identify the restaurant's name, and then generates a plausible, summarized review and rating for it.

 Core Logic and Architecture

The application is built using Next.js, React, and Genkit for the AI functionality. Here's a breakdown of how it works.

 1. Frontend and State Management (`src/app/page.tsx`)

The main page component manages the application's state and orchestrates the user flow. It uses React's `useState` hook to cycle through three primary states:

-   `'idle'`: The initial state. The `ImageUploader` component is displayed, prompting the user to upload a photo.
-   `'loading'`: Triggered after an image is selected. The `LoadingView` component is shown while the AI processes the image.
-   `'success'`: Occurs when the AI has successfully identified the restaurant and generated a review. The `ResultsView` component displays the outcome.

 2. Image Handling 

-   When a user uploads an image (either by clicking or drag-and-drop), the `handleImageUpload` function in `page.tsx` is called.
-   A `FileReader` is used to convert the image `File` object into a Base64-encoded **Data URI**. This string representation of the image is necessary to send the image data to the backend AI flow.

 3. AI Processing 

The core AI logic is handled by two Genkit flows located in `src/ai/flows/`.

 a. Restaurant Recognition 

-   This flow receives the image data URI from the frontend.
-   It uses a prompt that instructs the Gemini model to act as an expert in identifying restaurants from images.
-   The model analyzes the image to:
    1.  Determine if the image actually contains a restaurant storefront.
    2.  If it is a restaurant, extract the name using OCR and visual analysis.
-   It returns an object containing `isRestaurant` (a boolean) and `restaurantName` (a string).

b. Review Summarization 

-   If the first flow confirms a restaurant was found, this flow is called with the `restaurantName`.
-   It uses a prompt that instructs the model to act as a restaurant review analyst.
-   It generates a plausible and unique review summary, including:
    -   A star rating (1-5).
    -   A summary for **Food & Drinks**.
    -   A summary for **Service**.
    -   A summary for **Ambiance & Value**.
-   This flow includes a **retry mechanism** to handle potential API errors (like a 503 Service Unavailable), making the app more resilient.

 4. Scan History 

-   The `useHistory` custom hook provides a way to persist scan results.
-   When a review is successfully generated, the `addToHistory` function is called.
-   **Storage Quota Management**: To prevent `localStorage` from filling up (which would crash the app), the hook does not store the original full-resolution image. Instead, it creates a small **thumbnail** (64x64 JPEG) from the image data URI. This significantly reduces the storage footprint of each history item.
-   The history, containing these thumbnails, is saved to `localStorage` and displayed in the `HistorySidebar` component.

This architecture creates a seamless flow from image upload to displaying a unique, AI-generated review, providing a dynamic and interactive user experience.
