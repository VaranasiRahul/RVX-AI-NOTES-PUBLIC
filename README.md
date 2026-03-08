<div align="center">

# Daily Revision Hub
**AI-Powered Offline-First Study & Revision Application**

A React Native (Expo) mobile application designed to help users intelligently organize, summarize, and revise their notes using on-device Machine Learning (HuggingFace Transformers, ONNX) prioritizing privacy and speed.

*Note: This repository serves as a portfolio demonstration and architectural overview. The full source code is maintained in a private repository.*

</div>

## ✨ Feature Showcase

### 1. Smart Dashboard & Revision Feed
The central hub of the application. The **revision feed** intelligently surfaces note blocks that require attention. Using a liquid-glass aesthetic, the UI provides a distraction-free environment for focus.
<p align="center">
  <img src="assets/home_feed.png" height="450" />
</p>

### 2. On-Device AI Summarization
Experience zero-latency AI. Leveraging **HuggingFace Transformers** and **ONNX Runtime**, the app processes your raw notes locally to generate concise, actionable study blocks.
<p align="center">
  <img src="assets/ai_feed.png" height="450" />
  <img src="assets/ai_summary.png" height="450" />
</p>

### 3. Intelligent Organization & Topic Analysis
The app doesn't just store notes; it understands them. It automatically categorizes content into **Folders** and performs **Topic Analysis** to cluster related concepts together for deeper context.
<p align="center">
  <img src="assets/folder_org.png" height="450" />
  <img src="assets/topic_view.png" height="450" />
</p>

### 4. Focused Note Revision
Every generated block allows for deep dives. The **Note View** renders complex Markdown and allows users to jump back into the full context of their original documents instantly.
<p align="center">
  <img src="assets/note_detail.png" height="450" />
</p>

### 5. Habit Building & Personalization
Stay consistent with **Daily Streaks** and progress tracking. The **Advanced Settings** enable deep customization of the on-device AI models, UI theme (Midnight Glass, etc.), and local data management.
<p align="center">
  <img src="assets/streak.png" height="450" />
  <img src="assets/settings.png" height="450" />
</p>

---

## 🛠 Tech Stack
**Frontend (Mobile):**
- **React Native & Expo**: File-based routing with Expo Router.
- **Reanimated & Glassmorphism**: High-performance 60fps gestures and premium UI blurs.

**Edge AI Engine:**
- **@huggingface/transformers**: Local NLP processing.
- **ONNX Runtime**: Efficient on-device model execution.

**Persistence:**
- **Drizzle ORM**: Type-safe local SQLite interactions.
- **React Query**: Robust server-state and cache management.

---

## 🧠 System Architecture
To understand how the local AI interacts with the persistent storage and UI layer, please see the [Architecture Documentation](./ARCHITECTURE.md).

## 💡 Why This Project?
Built to solve the friction of manual revision, this app pushes AI models directly to the edge. It ensures **100% privacy** and **offline availability** while providing intelligent, adaptive study aids.

## 🔗 Contact & Links
- **LinkedIn**: [Your LinkedIn URL]
- **Portfolio**: [Your Portfolio URL]

---
*Created by [Your Name]*

