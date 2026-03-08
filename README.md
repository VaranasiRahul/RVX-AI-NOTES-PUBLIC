<div align="center">

# Daily Revision Hub
**AI-Powered Offline-First Study & Revision Application**

A React Native (Expo) mobile application designed to help users intelligently organize, summarize, and revise their notes using on-device Machine Learning (HuggingFace Transformers, ONNX) prioritizing privacy and speed.

*Note: This repository serves as a portfolio demonstration and architectural overview. The full source code is maintained in a private repository.*

</div>

## 📸 Screenshots & Demo

| Dashboard | AI Feed | AI Detail | Folder Org |
| :---: | :---: | :---: | :---: |
| <img src="assets/home_feed.png" height="350"> | <img src="assets/ai_feed.png" height="350"> | <img src="assets/ai_summary.png" height="350"> | <img src="assets/folder_org.png" height="350"> |
| **Topic List** | **Note View** | **Streak Tracking** | **Settings** |
| <img src="assets/topic_view.png" height="350"> | <img src="assets/note_detail.png" height="350"> | <img src="assets/streak.png" height="350"> | <img src="assets/settings.png" height="350"> |

---

## ✨ Key Features
- **On-Device AI Summarization**: Uses local ONNX models and `@huggingface/transformers` to generate document summaries without sending sensitive data to the cloud.
- **Smart Topic Extraction**: Automatically categorizes notes and extracts key topics for faster revision and folder organization.
- **Offline-First Architecture**: Features a robust local SQLite database using Drizzle ORM, ensuring all data is available without an active internet connection.
- **Persistence & Streaks**: Intelligent streak tracking and habit-building logic to encourage consistent daily revision.
- **Advanced Customization**: Comprehensive settings for AI model preferences, UI themes, and privacy-centric data management.
- **Story Mode**: A modern, immersive vertical-scrolling interface for reviewing saved note blocks (similar to social media stories).
- **Glassmorphism UI**: Premium visual aesthetics featuring liquid-glass navigation elements, dynamic blurs, and smooth Reanimated gestures.

## 🛠 Tech Stack
**Frontend (Mobile):**
- React Native & Expo (Expo Router for navigation)
- React Native Reanimated (Complex gesture and UI animations)
- Glass Effect & Blur (Liquid UI aesthetic)

**Local AI & Data Processing:**
- HuggingFace Transformers (`@huggingface/transformers`)
- ONNX Runtime (`onnxruntime-react-native`) for local embeddings & models

**Database & State:**
- Drizzle ORM (Local SQLite + server-side PostgreSQL integration capabilities)
- React Query (Data fetching, caching, and synchronization)
- Expo FileSystem (Persistent local document storage)

**Native Integrations:**
- React Native Android Widget (Kotlin/Java bridged widgets)
- Expo Location & Notifications

## 🧠 System Architecture
To understand how the local AI interacts with the persistent storage and UI layer, please see the [Architecture Documentation](./ARCHITECTURE.md).

## 💡 Why This Project?
This project was built to solve the pain point of scattered study notes and the friction of manually reviewing them. By pushing AI models directly to the edge (the mobile device), it ensures **zero latency** and **100% privacy** while providing intelligent, adaptive study aids.

## 🔗 Contact & Links
- **LinkedIn**: [Your LinkedIn URL]
- **Portfolio**: [Your Portfolio URL]

---
*Created by [Your Name]*
