# Chat

Chat is an innovative feature that allows users to interact with an AI-powered assistant within the context of a selected Zora profile. This feature enhances the user experience by providing personalized and context-aware responses.

## Getting Started

To start using the Chat, visit [https://chat.myco.wtf](https://chat.myco.wtf).

## How it Works

1. **Select a Zora Profile**: Choose a Zora profile to provide context for the conversation.
2. **Enter Your Intention**: Type in your question or topic of interest. Myco automatically adds relevant context using the Myco API.
3. **AI-Powered Response**: OpenAI's API is used to generate and stream a contextually appropriate response.

## Example Intents

- How many tokens has this artist created in the past week?
- How many different chains has this artist released on?
- What is their Zora score?
- Can you summarize this artist's recent activity on Zora?
- What is the most popular collection created by this artist?
- How does this artist's Zora score compare to the average?
- What themes or styles are prevalent in this artist's work?
- Has this artist collaborated with any other notable creators?
- What was the highest-grossing piece by this artist, and when was it sold?
- Can you provide insights on this artist's engagement with their community?

## Benefits

- **Personalized Interactions**: Get responses tailored to the context of a specific Zora profile.
- **Enhanced Understanding**: The AI assistant can provide more accurate and relevant information by considering the profile's context.
- **Streamlined Communication**: Easily obtain information about a creator's work, collections, or general blockchain topics.

## API Endpoint

The Chat feature utilizes the following Myco API endpoint to gather context:
`GET /api/chat/context?address={selected_profile_address}`

This endpoint fetches the context for the selected Zora profile, which includes the user's Zora Score, the list of tokens they have created, and other relevant information.
