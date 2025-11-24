<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Technical Solutions Document - Sun Life Claims Tracking</title>
    <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
            line-height: 1.6;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            background-color: white;
            padding: 40px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        h1 {
            color: #2c3e50;
            border-bottom: 3px solid #3498db;
            padding-bottom: 10px;
        }
        h2 {
            color: #34495e;
            margin-top: 30px;
            border-bottom: 2px solid #ecf0f1;
            padding-bottom: 8px;
        }
        h3 {
            color: #7f8c8d;
            margin-top: 20px;
        }
        .checkmark {
            color: #27ae60;
            font-weight: bold;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: left;
        }
        th {
            background-color: #3498db;
            color: white;
        }
        tr:nth-child(even) {
            background-color: #f9f9f9;
        }
        .mermaid {
            background-color: #fafafa;
            border: 1px solid #e0e0e0;
            border-radius: 4px;
            padding: 20px;
            margin: 20px 0;
        }
        code {
            background-color: #f4f4f4;
            padding: 2px 6px;
            border-radius: 3px;
            font-family: 'Courier New', monospace;
        }
        pre {
            background-color: #f4f4f4;
            padding: 15px;
            border-radius: 4px;
            overflow-x: auto;
        }
        .formula {
            background-color: #e8f4f8;
            padding: 15px;
            border-left: 4px solid #3498db;
            margin: 15px 0;
            font-family: monospace;
        }
        ul {
            line-height: 1.8;
        }
        .phase {
            background-color: #ecf0f1;
            padding: 15px;
            margin: 10px 0;
            border-radius: 4px;
        }
        hr {
            border: none;
            border-top: 2px solid #ecf0f1;
            margin: 30px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🧩 TECHNICAL SOLUTIONS DOCUMENT</h1>
        <h2>End-to-End Claim Tracking + True TAT + Accurate Counts</h2>
        <p><strong>Version 1.0</strong></p>

        <hr>

        <h2>1. Executive Summary</h2>
        <p>Sun Life receives:</p>
        <ul>
            <li><strong>Email notifications</strong> from 6 administrators</li>
            <li><strong>Claim files via FTP</strong>, which contain the actual claims</li>
        </ul>
        <p><strong>Current issues</strong> (inaccuracy, exceptions lost, wrong counts) arise because email notifications are being treated as claims.</p>
        <p>This document provides a fully accurate, auditable system for:</p>
        <ul>
            <li>Tracking the true number of claims received</li>
            <li>Tracking completed vs. exception cases</li>
            <li>Mapping BOT and manual processing</li>
            <li>Calculating true turnaround time (TAT)</li>
            <li>Automating Outlook email movement</li>
            <li>Creating 1:1 lifecycle tracking of every claim</li>
        </ul>

        <hr>

        <h2>2. Core Understandings</h2>
        <ul>
            <li>Emails are notifications, not claims</li>
            <li>FTP files are the true source of claims</li>
            <li>BOT results dictate Completed vs. Exception</li>
            <li>Email movement must happen AFTER the BOT report</li>
            <li>Meritain ZIP files must be expanded and tracked individually</li>
            <li>TAT requires full lifecycle timestamps (T0 → T3)</li>
        </ul>

        <hr>

        <h2>3. Lifecycle Timestamps (Required for True TAT)</h2>
        <table>
            <thead>
                <tr>
                    <th>Timestamp</th>
                    <th>Meaning</th>
                    <th>Source</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td><strong>T0</strong></td>
                    <td>Claim file arrived in FTP</td>
                    <td>FTP metadata / assignments </td>
                </tr>
                <tr>
                    <td><strong>T1</strong></td>
                    <td>File moved to BOT Action folder</td>
                    <td>BOT submission script</td>
                </tr>
                <tr>
                    <td><strong>T2</strong></td>
                    <td>BOT result time (Completed/Exception)</td>
                    <td>BOT morning report</td>
                </tr>
                <tr>
                    <td><strong>T3</strong></td>
                    <td>File successfully loaded into CMS</td>
                    <td>Data Intake Validation Report</td>
                </tr>
            </tbody>
        </table>

        <h3>Turnaround Time Formula</h3>
        <div class="formula">
            TAT = BusinessDays(T0 → T3)
        </div>

        <hr>

        <h2>4. High-Level System Overview</h2>
        <div class="mermaid">
flowchart TD
    A["Email Notifications<br>(Indicators Only)"] --> B["Inbox Tracking<br>Do NOT treat as claims"]
    C["FTP Claim Files<br>(Source of Truth)"] --> D["Assignment Job<br>Extract & Count Files"]
    D --> E[Assign FileID + T0]
    E --> F["Move Files to BOT Action Folder<br>Assign T1"]
    F --> G[BOT Overnight Processing]
    G --> H["BOT Morning Report<br>Completed | Exceptions | Pending"]
    
    H --> I["Completed Files<br>Move related emails to Completed Folder<br>Assign T2"]
    H --> J["Business Exceptions<br>Keep emails in Inbox<br>Assign T2"]
    H --> K["Pending<br>No movement"]
    
    I --> L["CMS Intake Validation<br>Assign T3"]
    J --> L
    L --> M[Final TAT + Accurate Counts]
        </div>

        <hr>

        <h2>5. FileID Strategy (Required for 1:1 Accuracy)</h2>
        <p>Every claim file receives a stable identifier:</p>
        <pre><code>&lt;Admin&gt;_&lt;RawFilename&gt;_&lt;YYYYMMDDHHMMSS&gt;_&lt;8CharHash&gt;</code></pre>

        <h3>Meritain ZIP Splitting</h3>
        <pre><code>ZIPID = &lt;Admin&gt;_&lt;ZipName&gt;_&lt;Timestamp&gt;
ChildID = &lt;ZIPID&gt;_&lt;InnerFileName&gt;_&lt;8CharHash&gt;</code></pre>
        <p>Each extracted file becomes its own claim with its own lifecycle.</p>

        <hr>

        <h2>6. Detailed Data Flow (Visual)</h2>
        <div class="mermaid">
sequenceDiagram
    participant E as Email Notification
    participant F as FTP Drop (Claim Files)
    participant A as Assignment Process
    participant ID as FileID Generator
    participant B as BOT Action Folder
    participant BOT as BOT Overnight
    participant R as BOT Morning Report
    participant O as Outlook Sync
    participant CMS as CMS Intake
    participant DB as Datastore (T0–T3)

    Note over E: Notification<br>Not a claim
    F->>A: New claim files detected
    A->>ID: Create FileID + Timestamp T0
    ID->>B: Move file to BOT Folder (T1)
    B->>BOT: BOT processes file
    BOT->>R: BOT Excel Report (Completed/Exception)
    R->>DB: Assign T2 for each FileID
    R->>O: Move Emails<br>Completed → Completed Folder<br>Exceptions → Stay in Inbox
    CMS->>DB: CMS Load Timestamp (T3)
    DB->>DB: Calculate TAT (BusinessDays)
        </div>

        <hr>

        <h2>7. Inbox Management Rules (Critical)</h2>
        
        <h3>Night Before:</h3>
        <ul>
            <li>Assignment job counts claims</li>
            <li>Files are moved to BOT Action folder</li>
            <li>Notification emails remain untouched</li>
        </ul>

        <h3>Morning BOT Report:</h3>
        <ul>
            <li><strong>Completed</strong> → move matching notifications to Completed folder</li>
            <li><strong>Exceptions</strong> → leave notifications in Inbox</li>
            <li><strong>Pending</strong> → no movement</li>
        </ul>

        <h3>Results:</h3>
        <ul>
            <li>Completed folder = true completed work</li>
            <li>Inbox = true exception inventory</li>
            <li>No double-counting</li>
            <li>Email counts finally match real BOT outcomes</li>
        </ul>

        <hr>

        <h2>8. Accurate Counts (Redefined)</h2>
        
        <h3>Count 1 — Claims Received (Accurate)</h3>
        <ul>
            <li><strong>Source:</strong> FTP → expanded ZIP contents → FileIDs</li>
            <li>Emails irrelevant.</li>
        </ul>

        <h3>Count 2 — Claims Completed (Accurate)</h3>
        <p><strong>Source:</strong></p>
        <ul>
            <li>BOT completed files</li>
            <li>Exceptions are NOT counted</li>
            <li>CMS validation ensures final completion</li>
        </ul>

        <h3>Count 3 — Email Notifications Received</h3>
        <ul>
            <li>Useful for trend analysis, but NOT claim volume.</li>
        </ul>

        <hr>

        <h2>9. Unified Datastore Structure</h2>
        <table>
            <thead>
                <tr>
                    <th>FileID</th>
                    <th>Admin</th>
                    <th>OriginalFile</th>
                    <th>ParentZIP</th>
                    <th>T0</th>
                    <th>T1</th>
                    <th>T2</th>
                    <th>T3</th>
                    <th>BOT Result</th>
                    <th>ManualTime</th>
                    <th>TAT</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td colspan="11" style="text-align: center; color: #7f8c8d;"><em>Data rows...</em></td>
                </tr>
            </tbody>
        </table>

        <p>This table powers:</p>
        <ul>
            <li>TAT calculations</li>
            <li>Exception tracking</li>
            <li>Admin-level performance</li>
            <li>Daily volume reporting</li>
            <li>BOT accuracy analysis</li>
        </ul>

        <hr>

        <h2>10. Complete Lifecycle Diagram</h2>
        <div class="mermaid">
flowchart LR
    A[FTP Claim Files] --> B[Assign FileID + T0]
    B --> C["Move to BOT Action Folder<br>Assign T1"]
    C --> D[BOT Processing Overnight]
    D --> E[BOT Morning Report]
    
    E --> F["Completed<br>Assign T2"]
    F --> G[Move related emails to Completed Folder]
    
    E --> H["Exceptions<br>Assign T2"]
    H --> I[Keep emails in Inbox]
    
    F --> J["CMS Load Validation<br>Assign T3"]
    H --> J
    
    J --> K["1:1 TAT Calculation<br>(BusinessDays T0→T3)"]
        </div>

        <hr>

        <h2>11. Final Output: TAT, Counts, and Accuracy</h2>
        <p>This system provides:</p>
        <ul>
            <li>True TAT based on FTP → CMS lifecycle</li>
            <li>Accurate daily claim volume based on FTP file count</li>
            <li>Accurate daily completions based on BOT + CMS</li>
            <li>Clear identification of all exceptions</li>
            <li>Automated Outlook inbox cleanup and integrity</li>
            <li>1:1 tracking for every claim file</li>
        </ul>

        <hr>

        <h2>12. Implementation Roadmap</h2>
        
        <div class="phase">
            <h3>Phase 1 — Foundation</h3>
            <ul>
                <li>FileID generator</li>
                <li>FTP extraction</li>
                <li>ZIP expansion</li>
                <li>Datastore creation</li>
            </ul>
        </div>

        <div class="phase">
            <h3>Phase 2 — BOT Automation</h3>
            <ul>
                <li>BOT report parser</li>
                <li>T1 / T2 tracking</li>
                <li>Exception flagging</li>
            </ul>
        </div>

        <div class="phase">
            <h3>Phase 3 — Outlook Automation</h3>
            <ul>
                <li>Email matching + movement script</li>
            </ul>
        </div>

        <div class="phase">
            <h3>Phase 4 — Full Picture TAT</h3>
            <ul>
                <li>T0–T3 population</li>
                <li>Business-day logic</li>
            </ul>
        </div>

        <div class="phase">
            <h3>Phase 5 — Reporting</h3>
            <ul>
                <li>Daily CSV</li>
                <li>Dashboard</li>
                <li>Other KPIs</li>
            </ul>
        </div>
    </div>

    <script>
        mermaid.initialize({ 
            startOnLoad: true,
            theme: 'default',
            flowchart: {
                useMaxWidth: true,
                htmlLabels: true,
                curve: 'basis'
            }
        });
    </script>
</body>
</html>
