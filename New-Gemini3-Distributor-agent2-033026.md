WOW Medical Device Distribution Analysis Studio
Comprehensive Technical Specification & Architecture Document
Document Version: 2.0.0
Date: March 30, 2026
Status: Approved for Implementation / Post-Deployment Review
Target Audience: Software Architects, Data Scientists, Compliance Officers, Product Managers, and Stakeholders.
1. Executive Summary
The WOW Medical Device Distribution Analysis Studio (MDDA Studio) is an enterprise-grade, browser-based analytical platform designed to revolutionize how medical device supply chains are audited, monitored, and optimized. Transitioning from legacy Python/Streamlit architectures to a modern, highly responsive React/Vite Single Page Application (SPA), this platform empowers compliance auditors, supply chain analysts, and business decision-makers to transform raw, unstructured distribution logs into actionable, AI-driven insights without writing a single line of code.
At its core, the system addresses the critical challenge of medical device traceability (e.g., tracking UDI-DIs, Lot Numbers, and Serial Numbers across complex distribution networks). By combining a robust client-side data processing pipeline, five highly advanced "WOW" interactive visualizations (powered by Apache ECharts), and a cutting-edge Agentic AI engine (powered by Google's Gemini 3.1 Pro), the MDDA Studio provides unprecedented visibility into supply chain concentration, temporal anomalies, and compliance risks.
This document serves as the definitive technical specification, detailing the system architecture, data models, visualization logic, AI integration strategies, and security protocols that underpin the platform.
2. System Architecture & Technology Stack
The MDDA Studio is architected as a fully client-side Single Page Application (SPA) to ensure maximum data privacy and minimal server-side infrastructure costs. All data parsing, standardization, and visualization rendering occur directly within the user's browser.
2.1 Core Technology Stack
Frontend Framework: React 19.0.0 with Vite 6.2.0 for lightning-fast Hot Module Replacement (HMR) and optimized production builds.
Language: TypeScript (Strict Mode) for end-to-end type safety, reducing runtime errors during complex data transformations.
Styling & UI Components: Tailwind CSS v4 integrated with shadcn/ui (Radix UI primitives) for a highly accessible, customizable, and responsive design system.
Data Parsing: PapaParse for high-performance, streaming CSV/TSV parsing, and native JSON.parse for structured data ingestion.
Visualization Engine: Apache ECharts (echarts-for-react and echarts-gl) for rendering complex, high-density datasets (Network Topologies, 3D Scatters) with WebGL acceleration.
AI/LLM Integration: @google/genai SDK for seamless, streaming communication with Gemini 3.1 Pro and Gemini 1.5 Flash models.
Configuration Management: js-yaml for parsing and serializing Agent configurations (agents.yaml).
2.2 Architectural Data Flow
Ingestion Layer: User uploads files or pastes raw text. The Data Ingestion Engine detects the format and parses it into an array of raw JavaScript objects.
Standardization Layer: The Canonical Schema Mapper iterates through the raw objects, applying heuristic rules to map disparate column names (e.g., Deliverdate, Date, deliver_date) to a strict CanonicalRecord interface.
State Management: The standardized data is stored in React's component state (useState). Derived datasets (e.g., filtered views, aggregated metrics) are memoized using useMemo to prevent unnecessary recalculations during re-renders.
Presentation Layer: The UI consumes the memoized data to render the Data Studio, WOW Dashboard, A/B Comparison, and Config Studio tabs.
AI Layer: When triggered, the system aggregates the canonical data into a high-density JSON summary and transmits it via secure API calls to the Gemini LLM, streaming the response back to the UI via react-markdown.
3. Core Modules & Feature Specifications
3.1 Global State & Top Navigation Bar
The application shell provides persistent global controls that affect the entire user experience.
Theme Engine: A toggleable Light/Dark mode that dynamically injects the .dark class into the HTML root, seamlessly integrated with Tailwind's dark mode variants.
Internationalization (i18n): A bilingual toggle (English / Traditional Chinese) that updates UI labels across all components in real-time using a lightweight dictionary object (t).
Painter Style Selector: A unique aesthetic feature offering 20 distinct color palettes inspired by famous painters (e.g., Van Gogh, Monet, Picasso). Selecting a style dynamically recalculates the HSL color arrays used by the ECharts rendering engine. A "Jackpot" (dice) button allows for randomized style selection.
Real-time Status Indicators: Badge components in the header display the current row count of the ingested dataset, the estimated LLM Token usage for the session, and the API connectivity status.
3.2 Data Ingestion & Preview Studio (DataStudio.tsx)
This module is responsible for safely loading and verifying user data.
Multi-Modal Ingestion:
File Upload: Utilizes the HTML5 File API to read .csv, .txt, and .json files asynchronously.
Clipboard Paste: A dedicated Textarea allows users to paste raw data directly from Excel or database dumps. The system employs a fallback parsing strategy (attempting JSON parse first, falling back to PapaParse for delimited text).
Dynamic Dual-View Preview:
To handle massive datasets without freezing the DOM, the preview table is virtualized/sliced based on a user-controlled slider (1 to 5000 records, default 20).
Raw Data Tab: Displays the unadulterated data exactly as ingested, allowing users to verify column headers and encoding.
Standardized Tab: Displays the data after it has passed through the Canonical Schema Mapper, highlighting how the system interpreted the raw inputs.
3.3 The WOW Dashboard (WowDashboard.tsx)
The crown jewel of the application, featuring five advanced, interactive visualizations built on Apache ECharts. The dashboard includes Global Filters (Supplier and Category) that instantly recalculate all five charts.
3.3.1 Distribution Network Graph (Force-Directed Topology)
Purpose: Visualizes the physical or logical flow of medical devices from upstream suppliers to downstream customers.
Data Structure: A 4-tier hierarchical network: Supplier → License Number → Device Model → Customer.
Rendering Logic: Uses ECharts graph type with force layout. Node sizes (symbolSize) are dynamically scaled based on the total quantity flowing through them. Edge widths (lineStyle.width) represent the volume of transactions between two specific nodes.
Interactivity: Nodes are draggable, and the physics engine (repulsion/edge length) recalculates in real-time.
3.3.2 Dynamic Time-Relation Infographic
Purpose: Tracks the expansion or contraction of the supply chain ecosystem over time.
Data Structure: A dual-axis combination chart. The X-axis represents the chronological deliver_date.
Rendering Logic:
Left Y-Axis (Lines): Tracks the distinct count of unique Suppliers and unique Device Models active on a given date.
Right Y-Axis (Bars): Tracks the distinct count of unique Customers receiving products on a given date.
Interactivity: Tooltips provide exact counts across all three metrics simultaneously for any given date.
3.3.3 3D Flow Scatter Matrix
Purpose: Identifies clustering, anomalies, and high-density trading periods in a three-dimensional space.
Data Structure: Utilizes echarts-gl for WebGL rendering.
X-Axis: Suppliers (Categorical)
Y-Axis: Customers (Categorical)
Z-Axis: Delivery Date (Chronological)
Rendering Logic: Data points are plotted in 3D space. The size of the scatter sphere represents the quantity of the transaction. The color mapping is tied to the selected "Painter Style".
Interactivity: Full 360-degree rotation, panning, and zooming via mouse/touch controls.
3.3.4 Compliance & Zone Heat Node Map (Treemap)
Purpose: Assesses risk concentration by visualizing how heavily certain licenses are distributed among specific customers.
Data Structure: Hierarchical data format. Parent nodes are License Numbers, and child nodes are Customers.
Rendering Logic: Uses the ECharts treemap type. The area of each rectangle is strictly proportional to the total quantity of devices distributed under that license to that specific customer.
Interactivity: Hovering reveals precise distribution volumes.
3.3.5 Bidirectional Hierarchy Chord Diagram
Purpose: Illustrates the complex, many-to-many relationships and dependencies between Suppliers and Customers.
Data Structure: A circular graph layout. Nodes represent both Suppliers and Customers arranged in a ring.
Rendering Logic: Edges (chords) connect Suppliers to Customers. The curvature and color of the edge indicate the directionality of the flow, while the thickness represents the volume (quantity).
3.4 A/B Dataset Deep Comparison (ABComparison.tsx)
Purpose: Allows analysts to perform side-by-side comparative analysis of two distinct subsets of the data (e.g., comparing the performance of Supplier A vs. Supplier B).
Implementation:
Two independent dropdown filters allow the user to define "Dataset A" and "Dataset B".
useMemo hooks isolate the filtered datasets.
Total Quantity Comparison: A side-by-side Bar Chart comparing the aggregate volume of the two datasets.
Category Distribution: Dual Pie Charts illustrating the proportional breakdown of device categories within each dataset, enabling quick identification of differing product focuses between suppliers.
3.5 Agentic AI Insight Engine (AgenticAI.tsx)
This module elevates the platform from a mere visualization tool to an intelligent analytical assistant, leveraging Google's Gemini 3.1 Pro model.
3.5.1 One-Click Comprehensive Report Generation
Data Aggregation Strategy: To prevent LLM context window overflow and reduce token costs, the raw dataset is not sent to the AI. Instead, a deterministic aggregation function calculates a high-density JSON summary containing: Total Records, Unique Entities (Suppliers, Customers, Models), Total Quantity, and a Top Suppliers volume map.
Prompt Engineering: A strict system prompt instructs the LLM to act as a "Medical Device Supply Chain Analyst and Compliance Auditor". It mandates a specific Markdown structure:
Executive Summary
Supply Chain Concentration Analysis (HHI evaluation)
Core Devices & License Distribution
Temporal Rhythm & Anomaly Insights
Compliance & Risk Assessment
Actionable Recommendations
Streaming Architecture: Utilizes generateContentStream to provide real-time, typewriter-effect feedback to the user, significantly reducing perceived latency for long-form (2000-3000 word) generation. The output is rendered safely using react-markdown.
3.5.2 Continuous Prompting (Chat Interface)
Purpose: Enables users to drill down into the generated report or ask ad-hoc questions about the dataset.
Implementation: Maintains a chatHistory array containing user and model roles. When a new question is asked, the entire conversation history is serialized and injected into the prompt context, ensuring the LLM maintains state awareness of previous analytical conclusions.
3.6 Config Studio & Intelligent YAML Governance (ConfigStudio.tsx)
Purpose: Manages the agents.yaml configuration file, which defines the behavior, models, and prompts for various AI personas.
Core Features: Upload, Download, and live Textarea editing of the YAML configuration.
Validation Engine: Uses js-yaml to parse the input. It enforces a strict schema requiring id, name, provider, and model fields.
AI-Driven Auto-Standardization: If validation fails (e.g., due to syntax errors, missing quotes, or incorrect indentation), the system exposes an "AI Standardization" button.
Mechanism: Sends the broken text to a fast, lightweight model (Gemini 3.1 Flash Lite) with a strict prompt to act as a "YAML architecture engineer". The LLM repairs the syntax, infers missing fields, and returns a clean, compliant YAML string, which is then automatically applied to the editor.
4. Data Dictionary & Canonical Schema
To ensure system stability, all ingested data must be mapped to the CanonicalRecord interface. The parseAndStandardize function employs regex and heuristic matching to handle common variations in raw column headers.
Canonical Field	Data Type	Description & Mapping Heuristics
supplier_id	String	The unique identifier for the distributor/supplier. Maps from SupplierID, supplier_id, or Supplier.
deliver_date	String	The date of transaction. Maps from Deliverdate, Date. Automatically formats YYYYMMDD to ISO YYYY-MM-DD.
customer_id	String	The unique identifier for the receiving hospital/client. Maps from CustomerID, Customer.
license_no	String	The official medical device license number. Maps from LicenseNo, License.
category	String	The classification of the device. Strips full-width quotes (“”) during parsing.
udi_di	String	Unique Device Identification - Device Identifier. Maps from UDID, UDI.
device_name	String	The commercial name of the product. Strips full-width quotes.
lot_no	String	The manufacturing lot or batch number. Critical for recall analysis. Maps from LotNO, Lot.
serial_no	String	The individual serial number of the device (if applicable).
model	String	The specific model number/code of the device.
quantity	Number	The volume of devices in the transaction. Maps from Number, Quantity. Defaults to 1 if unparseable.
5. Security, Privacy & Compliance Architecture
Given the sensitive nature of medical device distribution data (which may contain proprietary supply chain routes or indirectly identifiable hospital procurement volumes), the MDDA Studio is designed with a "Zero-Trust Server" architecture.
Client-Side Processing: 100% of the CSV/JSON parsing, data standardization, and chart rendering occurs within the user's local browser memory. No raw distribution data is ever transmitted to a backend database or server.
LLM Data Minimization: When utilizing the Agentic AI features, the system explicitly does not send the raw dataset to the Gemini API. Instead, it computes a mathematical and structural summary (counts, unique sets, aggregated volumes). This ensures that specific Serial Numbers, Lot Numbers, and exact transaction timestamps remain strictly on the client machine, mitigating the risk of data leakage to third-party LLM providers.
API Key Management: The GEMINI_API_KEY is injected via environment variables at build/runtime by the hosting platform (e.g., Google AI Studio / Cloud Run). The application never exposes UI fields asking users to input their own API keys, preventing phishing or accidental key exposure.
Stateless Architecture: The application does not use cookies, LocalStorage, or IndexedDB to persist medical data across sessions. Refreshing the browser completely wipes the memory, ensuring compliance with shared-workstation security policies.
6. Performance Optimization Strategies
Handling tens of thousands of rows of distribution data in a browser requires strict performance optimizations:
Memoization (useMemo): All derived datasets (e.g., extracting unique suppliers, filtering data for A/B comparison, aggregating nodes for the Network Graph) are wrapped in React useMemo hooks. These expensive calculations are only re-run when the underlying data or specific filters change, preventing UI stuttering during text input or theme toggling.
ECharts Canvas Rendering: Apache ECharts defaults to HTML5 <canvas> rendering, which is significantly more performant than SVG-based libraries (like D3.js) when dealing with thousands of DOM nodes (e.g., in the Network Topology and 3D Scatter plots).
Streaming Parsers: PapaParse is configured to skip empty lines and parse headers efficiently, allowing it to handle multi-megabyte CSV files in milliseconds without blocking the main JavaScript thread.
Debounced AI Rendering: The generateContentStream implementation updates the React state incrementally. While this causes frequent re-renders, the ReactMarkdown component is isolated within a ScrollArea to ensure the rest of the application remains responsive during generation.
7. Future Roadmap & Extensibility
While the current V2.0.0 release provides a comprehensive suite of tools, the architecture is designed to support future expansions:
Geospatial Mapping: Integration of echarts-leaflet or Mapbox to plot customer_id locations on a real-world map, provided the dataset is enriched with latitude/longitude coordinates.
Local LLM Support: Abstracting the @google/genai calls behind an interface to allow integration with local models (e.g., via WebGPU or Ollama) for environments with strict air-gapped compliance requirements.
Export Engine: Adding functionality to export the ECharts canvases as high-resolution PNG/PDFs, and the generated Markdown reports as formatted DOCX files for executive presentations.
8. 20 Comprehensive Follow-Up Questions for Stakeholders
To ensure continuous improvement and alignment with business objectives, the following 20 questions are posed to the project stakeholders, categorized by domain.
Data Governance & Schema Management
Schema Expansion: Currently, the CanonicalRecord defaults quantity to 1 if missing. Should we implement a strict validation mode that rejects records missing critical fields (like quantity or license_no) instead of attempting heuristic fixes?
UDI Compliance: The current system maps UDID to udi_di. Does the business require parsing of the UDI-PI (Production Identifier) to automatically extract expiration dates and manufacturing dates for deeper compliance analysis?
Data Appending: In the Data Studio, uploading a new file currently overwrites the existing dataset in memory. Should we implement an "Append" feature to allow users to stitch together multiple monthly CSV reports into a single session?
Handling Corrupt Dates: If the deliver_date heuristic fails to parse a non-standard date format (e.g., Julian dates), should the system drop the row, flag it in a dedicated "Error Log" UI, or attempt an LLM-based date correction?
UI/UX & Visualization Engine
Network Graph Density: For datasets exceeding 50,000 rows, the Force-Directed Network Graph may become visually cluttered. Should we implement a "Top N" pruning algorithm (e.g., only showing the top 80% of volume based on the Pareto principle) to maintain readability?
3D Scatter Usability: The 3D Flow Scatter Matrix is visually impressive but can be difficult to interpret for non-technical users. Should we add a feature to "flatten" it into a 2D heatmap via a toggle switch?
Painter Styles Accessibility: While the 20 Painter Styles add a "WOW" factor, some palettes may not meet WCAG 2.1 AA color contrast ratios. Should we introduce a "High Contrast / Colorblind Safe" toggle that overrides the artistic palettes?
Report Exporting: Users currently view the 3000-word AI report in the browser. What is the preferred format for exporting this report (e.g., PDF, Word, Markdown file) for sharing with external regulatory bodies?
AI & LLM Strategy
Token Cost Management: The "Comprehensive Report" utilizes Gemini 3.1 Pro, which has associated token costs. Should we implement a hard limit on how many reports a user can generate per session, or provide a "Lite Report" option using Gemini Flash?
Data Summary Density: Currently, we send aggregated counts and top suppliers to the LLM to avoid context overflow. Are there specific statistical metrics (e.g., standard deviation of delivery times, specific lot number tracking) that must be added to this JSON payload for the AI to analyze?
Agentic Actions: The AI currently only reads data and generates text. Should we grant the AI "Tool Calling" capabilities so it can automatically adjust the UI filters (e.g., User asks "Show me Boston Scientific", and the AI automatically changes the Supplier dropdown in the dashboard)?
Continuous Chat Context: The continuous prompting feature sends the entire chat history back to the LLM. For very long sessions, this will degrade performance. Should we implement a sliding window (e.g., only keeping the last 5 interactions) or a summarization step for the chat history?
Architecture & Performance
Web Worker Offloading: If users begin uploading 100MB+ CSV files, the main thread will block during PapaParse execution. Should we prioritize moving the parsing and canonical mapping logic into a dedicated Web Worker?
State Persistence: Currently, refreshing the page clears all data. Should we implement IndexedDB storage to save the parsed dataset locally, allowing users to resume their analysis after closing the browser tab?
ECharts Bundle Size: Importing echarts-gl significantly increases the JavaScript bundle size. If load times become an issue on slow hospital networks, should we lazy-load the 3D visualization tab only when the user clicks on it?
A/B Testing Scalability: The A/B Comparison currently compares two suppliers. Is there a business need to expand this to a multi-variate comparison (e.g., comparing 3 or 4 entities simultaneously)?
Security, Compliance & Deployment
Data Masking: Although data is processed client-side, the aggregated JSON sent to the LLM includes real supplier_id and customer_id values. Does company policy require us to cryptographically hash or pseudonymize these IDs before sending them to the Gemini API?
Audit Logging: For regulatory compliance (e.g., FDA 21 CFR Part 11), do we need to implement a mechanism that logs which user generated an AI report and what data was analyzed, transmitting this log to a secure backend?
YAML Security: The Config Studio allows users to upload custom agents.yaml files. While we validate the schema, should we implement a strict whitelist of allowed provider and model values to prevent users from configuring the system to call unauthorized external APIs?
Deployment Environment: The current app is optimized for Google Cloud Run / AI Studio. If the client requests an on-premise deployment within a hospital intranet, how will we handle the Gemini API dependency? Should we architect a fallback to a locally hosted model (e.g., Llama 3 via Ollama)?
