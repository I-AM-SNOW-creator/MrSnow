import streamlit as st
import json
import time
import os
from datetime import datetime
import psutil

LOG_FILE = "me_vs_me_activity_log.json"
REPORT_FILE = "me_vs_me_daily_report.json"

def track_activity(duration_minutes=1):
    activity_data = []
    start_time = time.time()
    end_time = start_time + (duration_minutes * 60)

    while time.time() < end_time:
        active_apps = {}
        for proc in psutil.process_iter(['pid', 'name']):
            try:
                name = proc.info['name']
                active_apps[name] = active_apps.get(name, 0) + 1
            except:
                continue
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        activity_data.append({'timestamp': timestamp, 'apps': active_apps})
        time.sleep(5)

    with open(LOG_FILE, 'w') as f:
        json.dump(activity_data, f, indent=2)

    return activity_data

def generate_daily_report(activity_data):
    app_usage = {}
    for entry in activity_data:
        for app, count in entry['apps'].items():
            app_usage[app] = app_usage.get(app, 0) + count

    sorted_apps = sorted(app_usage.items(), key=lambda x: x[1], reverse=True)
    total_entries = len(activity_data)

    report = {
        'date': datetime.now().strftime('%Y-%m-%d'),
        'total_checkpoints': total_entries,
        'top_apps': sorted_apps[:5]
    }

    with open(REPORT_FILE, 'w') as f:
        json.dump(report, f, indent=2)

    return report

def load_data():
    if os.path.exists(REPORT_FILE):
        with open(REPORT_FILE, 'r') as f:
            return json.load(f)
    return {}

# Streamlit App
st.title("ME VS ME – AI Mirror Challenge System")
st.write("Track your activity and challenge your past self daily.")

if st.button("Start 1-Minute Tracking Demo"):
    data = track_activity(1)
    report = generate_daily_report(data)
    st.success("Tracking complete! Report generated.")

report = load_data()
if report:
    st.subheader(f"Daily Report – {report['date']}")
    st.write(f"**Focus Checkpoints:** {report['total_checkpoints']}")
    st.write("**Top Apps:**")
    for app, count in report['top_apps']:
        st.markdown(f"- **{app}**: {count} times")
else:
    st.info("No report available yet. Run a demo to generate your first challenge.")
