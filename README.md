# Youtube-video-summary-Generator
A powerful web application that extracts transcripts from YouTube videos and generates AI-powered summaries using Google's Gemini AI. Built with Streamlit for an intuitive user experience.

## Features
```
🎥 YouTube URL Support - Works with various YouTube URL formats
📝 Automatic Transcript Extraction - Extracts video transcripts and captions
🤖 AI-Powered Summaries - Uses Google Gemini AI for intelligent summarization
📊 Clean Interface - User-friendly Streamlit web interface
📁 Download Options - Download both summaries and full transcripts
⚡ Real-time Processing - Live progress indicators
🔍 URL Validation - Automatic YouTube URL validation
```
## Supported YouTube URL Formats
```
https://www.youtube.com/watch?v=VIDEO_ID
https://youtu.be/VIDEO_ID
https://www.youtube.com/embed/VIDEO_ID
https://www.youtube.com/v/VIDEO_ID
```
## Requirements
```
Python 3.7 or higher
Google Gemini API Key (free from Google AI Studio)
Internet connection
```
## Installation
### Clone or download the project files

mkdir youtube-summarizer

cd youtube-summarizer

### Install required packages

pip install streamlit google-genai python-dotenv youtube-transcript-api

### Get your Google Gemini API Key
```
Go to Google AI Studio
Sign in with your Google account
Click "Get API Key" and create a new key
Copy the API key (starts with AIzaSy...)
```
### Create environment file

Create a .env file in your project folder:

GOOGLE_API_KEY=your_api_key_here

### Project Structure
```
youtube-summarizer/
├── app.py                 # Main application file
├── .env                   # Environment variables (API key)
├── requirements.txt       # Python dependencies
└── README.md             # This file
```
### File Contents
#### app.py
```
import streamlit as st
import os
import re
from urllib.parse import urlparse, parse_qs
from youtube_transcript_api import YouTubeTranscriptApi
from youtube_transcript_api.formatters import TextFormatter
from google import genai
from google.genai import types
from dotenv import load_dotenv
# Load environment variables
load_dotenv()
# Initialize Gemini client
client = genai.Client(api_key=os.getenv("GOOGLE_API_KEY"))
def extract_video_id(youtube_url):
    """
    Extract video ID from various YouTube URL formats
    """
    # Regular expression patterns for different YouTube URL formats
    patterns = [
        r'(?:youtube\.com\/watch\?v=|youtu\.be\/|youtube\.com\/embed\/|youtube\.com\/v\/)([^&\n?#]+)',
        r'youtube\.com\/watch\?.*v=([^&\n?#]+)',
    ]
    
    for pattern in patterns:
        match = re.search(pattern, youtube_url)
        if match:
            return match.group(1)
    
    return None
def get_video_transcript(video_id):
    """
    Get transcript for a YouTube video
    """
    try:
        # Get transcript
        transcript_list = YouTubeTranscriptApi().fetch(video_id)
        
        # Format transcript as plain text
        formatter = TextFormatter()
        transcript_text = formatter.format_transcript(transcript_list)
        
        return transcript_text, None
    except Exception as e:
        error_msg = str(e)
        if "No transcript found" in error_msg or "Transcript disabled" in error_msg:
            return None, "No transcript available for this video. The video may not have captions enabled."
        elif "Video unavailable" in error_msg:
            return None, "Video is unavailable or private."
        else:
            return None, f"Error retrieving transcript: {error_msg}"
def generate_summary(transcript_text):
    """
    Generate AI-powered summary using Google Gemini
    """
    try:
        prompt = f"""
        Please provide a comprehensive summary of the following YouTube video transcript. 
        Focus on the main points, key insights, and important information discussed in the video.
        Make the summary clear, concise, and well-structured with bullet points for key topics.
        
        Transcript:
        {transcript_text}
        """
        
        response = client.models.generate_content(
            model="gemini-2.5-flash",
            contents=prompt
        )
        
        return response.text if response.text else "Failed to generate summary."
    except Exception as e:
        return f"Error generating summary: {str(e)}"
def get_video_title(video_id):
    """
    Extract video title by attempting to get transcript metadata
    """
    try:
        # Try to get transcript which sometimes includes video metadata
        transcript_list = YouTubeTranscriptApi().list(video_id)
        # This is a simple approach - in a real app you might use YouTube Data API
        return f"Video ID: {video_id}"
    except:
        return f"Video ID: {video_id}"
def validate_youtube_url(url):
    """
    Validate if the provided URL is a valid YouTube URL
    """
    youtube_domains = ['youtube.com', 'youtu.be', 'www.youtube.com', 'm.youtube.com']
    
    try:
        parsed_url = urlparse(url)
        return parsed_url.netloc.lower() in youtube_domains
    except:
        return False
def main():
    # Page configuration
    st.set_page_config(
        page_title="YouTube Video Summarizer",
        page_icon="📺",
        layout="wide"
    )
    
    # Main title and description
    st.title("📺 YouTube Video Summarizer")
    st.markdown("Extract transcripts from YouTube videos and generate AI-powered summaries using Google's Generative AI.")
    
    # Sidebar with instructions
    with st.sidebar:
        st.header("How to use:")
        st.markdown("""
        1. **Paste YouTube URL** - Copy any YouTube video URL
        2. **Click Summarize** - The app will extract the transcript
        3. **Get Summary** - AI will generate a comprehensive summary
        
        **Supported URL formats:**
        - youtube.com/watch?v=VIDEO_ID
        - youtu.be/VIDEO_ID
        - youtube.com/embed/VIDEO_ID
        """)
        
        st.header("Note:")
        st.info("Only videos with available transcripts/captions can be summarized.")
    
    # Main content area
    col1, col2 = st.columns([2, 1])
    
    with col1:
        # URL input
        st.subheader("Enter YouTube Video URL")
        youtube_url = st.text_input(
            "YouTube URL:",
            placeholder="https://www.youtube.com/watch?v=dQw4w9WgXcQ",
            help="Paste the URL of the YouTube video you want to summarize"
        )
        
        # Summarize button
        summarize_button = st.button(
            "🔍 Summarize Video", 
            type="primary",
            use_container_width=True
        )
    
    with col2:
        # Status indicators
        if youtube_url:
            if validate_youtube_url(youtube_url):
                st.success("✅ Valid YouTube URL")
            else:
                st.error("❌ Invalid YouTube URL")
    
    # Process video when button is clicked
    if summarize_button:
        if not youtube_url:
            st.error("Please enter a YouTube URL.")
            return
        
        if not validate_youtube_url(youtube_url):
            st.error("Please enter a valid YouTube URL.")
            return
        
        # Extract video ID
        video_id = extract_video_id(youtube_url)
        if not video_id:
            st.error("Could not extract video ID from the URL. Please check the URL format.")
            return
        
        # Show video information
        st.subheader("Video Information")
        video_title = get_video_title(video_id)
        st.info(f"**Video:** {video_title}")
        
        # Progress indicator
        progress_bar = st.progress(0)
        status_text = st.empty()
        
        # Step 1: Extract transcript
        status_text.text("Extracting transcript...")
        progress_bar.progress(33)
        
        transcript_text, error = get_video_transcript(video_id)
        
        if error:
            st.error(f"❌ {error}")
            return
        
        if not transcript_text:
            st.error("No transcript found for this video.")
            return
        
        # Step 2: Generate summary
        status_text.text("Generating AI summary...")
        progress_bar.progress(66)
        
        summary = generate_summary(transcript_text)
        
        # Step 3: Complete
        status_text.text("Complete!")
        progress_bar.progress(100)
        
        # Display results
        st.success("✅ Summary generated successfully!")
        
        # Create tabs for transcript and summary
        tab1, tab2 = st.tabs(["📝 AI Summary", "📄 Full Transcript"])
        
        with tab1:
            st.subheader("AI-Generated Summary")
            st.markdown(summary)
            
            # Download summary button
            st.download_button(
                label="📥 Download Summary",
                data=summary,
                file_name=f"youtube_summary_{video_id}.txt",
                mime="text/plain"
            )
        
        with tab2:
            st.subheader("Full Transcript")
            with st.expander("View Full Transcript", expanded=False):
                st.text_area(
                    "Transcript:",
                    value=transcript_text,
                    height=400,
                    disabled=True
                )
            
            # Download transcript button
            st.download_button(
                label="📥 Download Transcript",
                data=transcript_text,
                file_name=f"youtube_transcript_{video_id}.txt",
                mime="text/plain"
            )
        
        # Clear progress indicators
        progress_bar.empty()
        status_text.empty()
    
    # Footer
    st.markdown("---")
    st.markdown(
        """
        <div style='text-align: center; color: #666;'>
            <p>Powered by Google Generative AI and YouTube Transcript API</p>
        </div>
        """,
        unsafe_allow_html=True
    )
if __name__ == "__main__":
    main()
```
### .env
```
GOOGLE_API_KEY=your_actual_api_key_here
```
### requirements.txt
```
streamlit
google-genai
python-dotenv
youtube-transcript-api
```
### How to Use
### Start the application

streamlit run app.py
### Open your browser

The app will automatically open at http://localhost:8501
If it doesn't open automatically, visit the URL shown in the terminal
Use the application

### Paste any YouTube video URL in the input field
Click "🔍 Summarize Video"
Wait for the transcript extraction and AI summary generation
View results in the "AI Summary" and "Full Transcript" tabs
Download summaries or transcripts as needed

## outputs
<img width="1763" height="781" alt="image" src="https://github.com/user-attachments/assets/e43faf2e-e3c9-47cd-a9c0-f99d14f77816" />
<img width="1880" height="816" alt="image" src="https://github.com/user-attachments/assets/c3b264c2-b931-4c63-a22b-65bf4170f10e" />
<img width="1747" height="865" alt="image" src="https://github.com/user-attachments/assets/a6d55325-7be6-40f9-90b9-1484f4f3470e" />


## Result
After running the program with a YouTube video link, the application successfully extracts the transcript and generates summaries at multiple levels.
