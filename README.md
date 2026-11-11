import streamlit as st

# Page Configuration
st.set_page_config(
    page_title="COD VOD Coach",
    page_icon="🎯",
    layout="wide"
)

# Custom Styling for Dark Tactical Theme
st.markdown("""
    <style>
    .main {
        background-color: #0e1117;
        color: #ffffff;
    }
    .stTextInput input {
        background-color: #1a1c23;
        color: #ffffff;
        border: 1px solid #30363d;
        border-radius: 8px;
        padding: 12px;
    }
    .stButton button {
        width: 100%;
        background-color: #ff4b4b;
        color: white;
        font-weight: bold;
        border-radius: 8px;
        padding: 10px;
        border: none;
    }
    .stButton button:hover {
        background-color: #ff2b2b;
    }
    .blueprint-card {
        background-color: #161b22;
        border: 1px solid #30363d;
        padding: 15px;
        border-radius: 8px;
        margin-bottom: 10px;
    }
    </style>
""", unsafe_allow_html=True)

# Initialize Session State
if "analyzed" not in st.session_state:
    st.session_state.analyzed = False
if "video_url" not in st.session_state:
    st.session_state.video_url = ""
if "messages" not in st.session_state:
    st.session_state.messages = []
if "blueprints" not in st.session_state:
    # Default sample blueprint
    st.session_state.blueprints = [{
        "gun": "Holger 556 / Meta AR",
        "attachments": [
            "Muzzle: VT-7 Spitfire Suppressor",
            "Barrel: Chrios Match-Grade",
            "Underbarrel: Bruen Heavy Support Grip",
            "Magazine: 40 Round Mag",
            "Optic: JAK Glassless Optic"
        ]
    }]

# --- VIEW 1: HOME PAGE WITH TABS ---
if not st.session_state.analyzed:
    st.markdown("<h1 style='text-align: center; margin-bottom: 20px;'>🎯 COD VOD COACH</h1>", unsafe_allow_html=True)
    
    tab_analysis, tab_targets = st.tabs(["🚀 VOD Analysis", "📊 Gameplay Targets"])
    
    with tab_analysis:
        st.markdown("<p style='text-align: center; color: #8b949e;'>Drop your match footage URL below to begin coaching analysis.</p>", unsafe_allow_html=True)
        
        col1, col2, col3 = st.columns([1, 2, 1])
        with col2:
            with st.form(key='url_form'):
                video_url_input = st.text_input(
                    label="Footage Source",
                    placeholder="Paste YouTube URL or file path...",
                    label_visibility="collapsed"
                )
                submit_button = st.form_submit_button(label="Enter Analysis Suite")

            if submit_button:
                if video_url_input:
                    st.session_state.video_url = video_url_input
                    st.session_state.analyzed = True
                    st.session_state.messages.append({
                        "role": "assistant",
                        "content": f"Loaded footage (`{video_url_input}`). I'm ready to review your gameplay. Ask about positioning, centering, gunfights, or request a meta gun build."
                    })
                    st.rerun()
                else:
                    st.warning("Please enter a valid YouTube URL or file path.")
                    
    with tab_targets:
        st.markdown("### Core Gameplay Benchmarks & Targets")
        st.markdown("Use these baseline targets to evaluate your performance during VOD reviews:")
        
        col_t1, col_t2 = st.columns(2)
        with col_t1:
            st.markdown("""
            #### 🎯 Mechanical Targets
            * **Crosshair Centering:** Keep crosshairs locked at head/upper chest height constantly—zero floor-gazing.
            * **Pre-Aim Discipline:** Pre-aim common door frames and power positions *before* breaking cover, not after.
            * **Recoil Control:** Maintain >85% shot clustering within standard engagement ranges.
            """)
        with col_t2:
            st.markdown("""
            #### 🧠 Tactical Targets
            * **Cover Utilization:** Never take open-field gunfights without hard cover or heady-ready angles within sliding distance.
            * **Engagement Win-Rate:** Target a 60%+ win rate on initial 1v1 engagements by forcing right-hand peek advantages.
            * **Rotation Timing:** Initiate rotations 15 seconds before zone shifts or objective changes to secure power positions.
            """)

# --- VIEW 2: COACHING CHAT INTERFACE & BLUEPRINT SIDEBAR ---
else:
    # Sidebar for Saved Gun Blueprints
    with st.sidebar:
        st.markdown("### 🛠️ Saved Gun Blueprints")
        st.markdown("Quick-reference loadouts recommended during coaching sessions:")
        st.divider()
        
        if not st.session_state.blueprints:
            st.info("No blueprints saved yet. Ask the coach for a gun build!")
        else:
            for bp in st.session_state.blueprints:
                st.markdown(f"""
                <div class="blueprint-card">
                    <strong>{bp['gun']}</strong>
                    <ul style="padding-left: 15px; margin-top: 5px; font-size: 0.85em; color: #c9d1d9;">
                """, unsafe_allow_html=True)
                for att in bp['attachments']:
                    st.markdown(f"<li>{att}</li>", unsafe_allow_html=True)
                st.markdown("</ul></div>", unsafe_allow_html=True)
                
        st.divider()
        if st.button("🔄 New Footage"):
            st.session_state.analyzed = False
            st.session_state.video_url = ""
            st.session_state.messages = []
            st.rerun()

    # Main Chat Area Header
    st.markdown(f"**Active VOD:** `{st.session_state.video_url}`")
    st.divider()

    # Display chat history
    for message in st.session_state.messages:
        with st.chat_message(message["role"]):
            st.markdown(message["content"])

    # Chat input box
    if prompt := st.chat_input("Ask about positioning, request gun builds, or analyze errors..."):
        st.session_state.messages.append({"role": "user", "content": prompt})
        with st.chat_message("user"):
            st.markdown(prompt)

        # Logic for handling gun build requests vs standard coaching
        prompt_lower = prompt.lower()
        if "gun" in prompt_lower or "build" in prompt_lower or "loadout" in prompt_lower or "meta" in prompt_lower:
            response = "Here is the recommended meta build for stability and fast time-to-kill. I've added it to your sidebar blueprint reference for easy access:"
            new_bp = {
                "gun": "MCW / Balanced Assault Rifle",
                "attachments": [
                    "Muzzle: Casus Brake",
                    "Barrel: 16.5\" MCW Cyclone Long Barrel",
                    "Underbarrel: DR-6 Handstop",
                    "Rear Grip: RB Regal Grip",
                    "Stock: Precision Stock"
                ]
            }
            # Avoid duplicate entries
            if not any(b['gun'] == new_bp['gun'] for b in st.session_state.blueprints):
                st.session_state.blueprints.append(new_bp)
        else:
            response = f"Reviewing your VOD regarding '{prompt}': Focus heavily on your corner-clearing technique. You are exposing your left shoulder before checking the hard angle."

        st.session_state.messages.append({"role": "assistant", "content": response})
        with st.chat_message("assistant"):
            st.markdown(response)
            if "blueprint" in locals() and "new_bp" in locals():
                st.markdown(f"**Blueprint Added:** `{new_bp['gun']}` (Check sidebar)")
                st.rerun()
                
