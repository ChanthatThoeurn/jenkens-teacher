
// pipeline {
//     agent any

//     stages {
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Build') {
//             steps {
//                 sh 'docker build -t jenkins-react-pipeline .'
//             }
//         }

//         stage('Deploy') {
//             steps {
//                 sh '''
//                     set -eu

//                     echo "Step 1: Test the new image..."

//                     docker rm -f reactjs-test 2>/dev/null || true

//                     docker run -d \
//                       --name reactjs-test \
//                       -p 127.0.0.1:3006:3000 \
//                       jenkins-react-pipeline

//                     sleep 5

//                     if ! curl -fsS http://127.0.0.1:3006/ > /dev/null; then
//                         echo "New image failed health check!"
//                         docker logs reactjs-test
//                         docker rm -f reactjs-test
//                         exit 1
//                     fi

//                     echo "New image passed health check."

//                     docker rm -f reactjs-test

//                     echo "Step 2: Preserve the old container..."

//                     docker rm -f reactjs-old 2>/dev/null || true

//                     if docker container inspect reactjs-cont >/dev/null 2>&1; then
//                         docker stop reactjs-cont
//                         docker rename reactjs-cont reactjs-old
//                     fi

//                     echo "Step 3: Start the new application..."

//                     if ! docker run -d \
//                       --name reactjs-cont \
//                       --restart unless-stopped \
//                       -p 3005:3000 \
//                       jenkins-react-pipeline; then

//                         echo "New container failed. Rolling back..."

//                         docker rm -f reactjs-cont 2>/dev/null || true
//                         docker rename reactjs-old reactjs-cont
//                         docker start reactjs-cont
//                         exit 1
//                     fi

//                     sleep 5

//                     if ! curl -fsS http://127.0.0.1:3005/ > /dev/null; then
//                         echo "New application failed. Rolling back..."

//                         docker logs reactjs-cont
//                         docker rm -f reactjs-cont

//                         docker rename reactjs-old reactjs-cont
//                         docker start reactjs-cont

//                         exit 1
//                     fi

//                     echo "Deployment successful!"

//                     docker rm -f reactjs-old 2>/dev/null || true
//                 '''
//             }
//         }
//     }
// }
// Jenkinsfile for text-zap: Docker build, push, deploy, ZAP, Telegram + Email
// def esc(String s) {

//     return (s ?: '').replace('&', '&amp;').replace('<', '&lt;').replace('>', '&gt;')

// }

// def sendTelegram(String status) {

//     boolean ok = (status == 'SUCCESS')

//     String icon  = ok ? '✅' : '❌'

//     String title = ok ? 'Deployment Successful' : 'Deployment Failed'

//     String commit = 'n/a'

//     String author = 'n/a'

//     try {

//         commit = sh(script: 'git log -1 --pretty=%s', returnStdout: true).trim()

//         author = sh(script: 'git log -1 --pretty=%an', returnStdout: true).trim()

//     } catch (err) {

//         echo 'Could not read git info'

//     }

//     String now = sh(script: "date '+%d %b %Y, %H:%M %Z'", returnStdout: true).trim()

//     String duration = currentBuild.durationString.replace(' and counting', '')

//     String msg = """${icon} <b>${title}</b>

// ━━━━━━━━━━━━━━━

// 📦 <b>Project:</b>  <code>${esc(env.JOB_NAME)}</code>

// 🔢 <b>Build:</b>  #${env.BUILD_NUMBER}

// 🐳 <b>Image:</b>  <code>${esc(env.DOCKERHUB_REPO)}:${env.IMAGE_TAG}</code>

// 👤 <b>Author:</b>  ${esc(author)}

// 💬 <b>Commit:</b>  ${esc(commit)}

// ⏱ <b>Duration:</b>  ${duration}

// 🕐 <b>Time:</b>  ${now}"""

//     String buttons = ok

//         ? """{"inline_keyboard":[[{"text":"🔍 Build","url":"${env.BUILD_URL}"},{"text":"📜 Console","url":"${env.BUILD_URL}console"}],[{"text":"🌐 Open App","url":"http://3.233.255.129:3005"}]]}"""

//         : """{"inline_keyboard":[[{"text":"🔍 Build","url":"${env.BUILD_URL}"},{"text":"📜 Console","url":"${env.BUILD_URL}console"}]]}"""

//     writeFile file: 'tg_msg.txt', text: msg

//     writeFile file: 'tg_buttons.json', text: buttons

//     withCredentials([

//         string(credentialsId: 'telegram-bot-token', variable: 'TG_TOKEN'),

//         string(credentialsId: 'telegram-chat-id', variable: 'TG_CHAT')

//     ]) {

//         sh '''

//             curl -s -X POST "https://api.telegram.org/bot${TG_TOKEN}/sendMessage" \\
//               -d chat_id="${TG_CHAT}" \\
//               -d parse_mode="HTML" \\
//               -d disable_web_page_preview=true \\
//               --data-urlencode "text@tg_msg.txt" \\
//               --data-urlencode "reply_markup@tg_buttons.json" \\
//               || true

//             rm -f tg_msg.txt tg_buttons.json

//         '''

//     }

// }

// def sendEmail(String status) {

//     boolean ok = (status == 'SUCCESS')

//     String icon    = ok ? '✅' : '❌'

//     String title   = ok ? 'Deployment Successful' : 'Deployment Failed'

//     String sub     = ok ? 'Your app is live and healthy.' : 'Something went wrong. Check the console log.'

//     String color   = ok ? '#16a34a' : '#dc2626'

//     String soft    = ok ? '#dcfce7' : '#fee2e2'

//     String commit = 'n/a'

//     String author = 'n/a'

//     try {

//         commit = sh(script: 'git log -1 --pretty=%s', returnStdout: true).trim()

//         author = sh(script: 'git log -1 --pretty=%an', returnStdout: true).trim()

//     } catch (err) {

//         echo 'Could not read git info'

//     }

//     String now = sh(script: "date '+%d %b %Y, %H:%M %Z'", returnStdout: true).trim()

//     String duration = currentBuild.durationString.replace(' and counting', '')

//     String appButton = ok

//         ? """<a href="http://3.233.255.129:3005" style="display:inline-block;padding:12px 22px;margin:4px;background:#0f172a;color:#ffffff;text-decoration:none;border-radius:8px;font-weight:600;font-size:14px;">🌐 Open App</a>"""

//         : ''

//     String html = """<!DOCTYPE html>

// <html>

// <body style="margin:0;padding:0;background:#f1f5f9;font-family:Segoe UI,Roboto,Helvetica,Arial,sans-serif;">

//   <table width="100%" cellpadding="0" cellspacing="0" style="background:#f1f5f9;padding:24px 0;">

//     <tr><td align="center">

//       <table width="600" cellpadding="0" cellspacing="0" style="max-width:600px;width:100%;background:#ffffff;border-radius:14px;overflow:hidden;box-shadow:0 4px 18px rgba(15,23,42,0.10);">

//         <tr><td style="background:${color};padding:32px 28px;text-align:center;">

//           <div style="font-size:44px;line-height:1;">${icon}</div>

//           <div style="color:#ffffff;font-size:26px;font-weight:700;margin-top:10px;">${title}</div>

//           <div style="color:#ffffff;opacity:0.9;font-size:14px;margin-top:6px;">${sub}</div>

//         </td></tr>

//         <tr><td style="padding:26px 28px 8px 28px;">

//           <div style="display:inline-block;background:${soft};color:${color};font-weight:700;font-size:12px;letter-spacing:1px;padding:5px 12px;border-radius:999px;">${status}</div>

//           <div style="font-size:20px;font-weight:700;color:#0f172a;margin-top:12px;">${esc(env.JOB_NAME)} <span style="color:#64748b;font-weight:500;">#${env.BUILD_NUMBER}</span></div>

//         </td></tr>

//         <tr><td style="padding:12px 28px 8px 28px;">

//           <table width="100%" cellpadding="0" cellspacing="0" style="font-size:14px;border-collapse:collapse;">

//             <tr>

//               <td style="padding:12px 0;border-bottom:1px solid #e2e8f0;color:#64748b;width:130px;">🐳 Image</td>

//               <td style="padding:12px 0;border-bottom:1px solid #e2e8f0;color:#0f172a;font-family:Consolas,Menlo,monospace;">${esc(env.DOCKERHUB_REPO)}:${env.IMAGE_TAG}</td>

//             </tr>

//             <tr>

//               <td style="padding:12px 0;border-bottom:1px solid #e2e8f0;color:#64748b;">👤 Author</td>

//               <td style="padding:12px 0;border-bottom:1px solid #e2e8f0;color:#0f172a;">${esc(author)}</td>

//             </tr>

//             <tr>

//               <td style="padding:12px 0;border-bottom:1px solid #e2e8f0;color:#64748b;">💬 Commit</td>

//               <td style="padding:12px 0;border-bottom:1px solid #e2e8f0;color:#0f172a;">${esc(commit)}</td>

//             </tr>

//             <tr>

//               <td style="padding:12px 0;border-bottom:1px solid #e2e8f0;color:#64748b;">⏱ Duration</td>

//               <td style="padding:12px 0;border-bottom:1px solid #e2e8f0;color:#0f172a;">${duration}</td>

//             </tr>

//             <tr>

//               <td style="padding:12px 0;color:#64748b;">🕐 Time</td>

//               <td style="padding:12px 0;color:#0f172a;">${now}</td>

//             </tr>

//           </table>

//         </td></tr>

//         <tr><td style="padding:20px 28px 30px 28px;text-align:center;">

//           <a href="${env.BUILD_URL}" style="display:inline-block;padding:12px 22px;margin:4px;background:${color};color:#ffffff;text-decoration:none;border-radius:8px;font-weight:600;font-size:14px;">🔍 View Build</a>

//           <a href="${env.BUILD_URL}console" style="display:inline-block;padding:12px 22px;margin:4px;background:#475569;color:#ffffff;text-decoration:none;border-radius:8px;font-weight:600;font-size:14px;">📜 Console Log</a>

//           ${appButton}

//         </td></tr>

//         <tr><td style="background:#f8fafc;padding:16px 28px;text-align:center;color:#94a3b8;font-size:12px;border-top:1px solid #e2e8f0;">

//           Sent automatically by Jenkins CI/CD

//         </td></tr>

//       </table>

//     </td></tr>

//   </table>

// </body>

// </html>"""

//     try {

//         emailext(

//             to: 'kokazono18@gmail.com',

//             subject: "${icon} ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${title}",

//             mimeType: 'text/html',

//             body: html,

//             attachLog: !ok

//         )

//     } catch (err) {

//         echo "Email failed: ${err.getMessage()}"

//     }

// }

// pipeline {

//     agent any

//     options {

//         timestamps()

//         timeout(time: 15, unit: 'MINUTES')

//         disableConcurrentBuilds()

//         buildDiscarder(logRotator(numToKeepStr: '15'))

//     }

//     environment {

//     DOCKERHUB_REPO = 'chanthat/reactjs-devop8-template'

//     IMAGE_TAG = "${BUILD_NUMBER}"

//     }

//     stages {

//         stage('1. Build Docker Image') {

//             steps {

//                 echo 'Building React Docker image...'

//                 sh '''

//                     docker build \\
//                       -t $DOCKERHUB_REPO:$IMAGE_TAG \\
//                       -t $DOCKERHUB_REPO:latest .

//                 '''

//             }

//         }

//         stage('2. Push to Docker Hub') {

//             steps {

//                 echo 'Pushing image to Docker Hub...'

//                 withCredentials([

//                     usernamePassword(

//                         credentialsId: 'dockerhub-credentials',

//                         usernameVariable: 'DOCKER_USER',

//                         passwordVariable: 'DOCKER_TOKEN'

//                     )

//                 ]) {

//                     sh '''

//                         echo "$DOCKER_TOKEN" | docker login \\
//                           -u "$DOCKER_USER" \\
//                           --password-stdin

//                         docker push $DOCKERHUB_REPO:$IMAGE_TAG

//                         docker push $DOCKERHUB_REPO:latest

//                         docker logout

//                     '''

//                 }

//             }

//         }

//         // stage('3. Deploy to EC2') {

//         //     when {

//         //         expression {

//         //             def branch = env.BRANCH_NAME ?: env.GIT_BRANCH ?: 'main'

//         //             return branch == 'main' || branch == 'origin/main'

//         //         }

//         //     }

//         //     steps {

//         //         echo 'Deploying React application...'

//         //         sh '''

//         //             docker pull $DOCKERHUB_REPO:$IMAGE_TAG

//         //             docker rm -f reactjs-cont 2>/dev/null || true

//         //             docker run -d \\
//         //               --name reactjs-cont \\
//         //               --restart unless-stopped \\
//         //               -p 3005:3000 \\
//         //               $DOCKERHUB_REPO:$IMAGE_TAG

//         //             sleep 5

//         //             curl -f http://127.0.0.1:3005/ \\
//         //               || (docker logs reactjs-cont && exit 1)

//         //         '''

//         //     }

//         // }

//         stage('4. OWASP ZAP Dynamic Security Scan') {
//             steps {
//                 script {
//                     echo 'Scanning https://thatdevops.duckdns.org/ with OWASP ZAP Baseline...'

//                     int zapStatus = sh(
//                         script: '''#!/bin/sh
//                             docker run --rm \\
//                               ghcr.io/zaproxy/zaproxy:stable \\
//                               zap-baseline.py \\
//                               -t https://thatdevops.duckdns.org/ \\
//                               -m 1 -s
//                         ''',
//                         returnStatus: true
//                     )

//                     echo "OWASP ZAP exit code: ${zapStatus}"

//                     if (zapStatus == 0) {
//                         echo 'ZAP PASSED: no policy failures or warnings.'
//                     } else if (zapStatus == 1 || zapStatus == 2) {
//                         echo 'ZAP detected security findings; sending Telegram alert.'
//                         withCredentials([
//                             string(credentialsId: 'telegram-bot-token', variable: 'TG_TOKEN'),
//                             string(credentialsId: 'telegram-chat-id', variable: 'TG_CHAT')
//                         ]) {
//                             withEnv(["ZAP_ALERT=OWASP ZAP found security findings (exit ${zapStatus}) on https://thatdevops.duckdns.org/ | Jenkins: ${env.BUILD_URL}"]) {
//                                 sh '''#!/bin/sh
//                                     set +x
//                                     curl --fail --silent --show-error \\
//                                       --request POST \\
//                                       "https://api.telegram.org/bot${TG_TOKEN}/sendMessage" \\
//                                       --data-urlencode "chat_id=${TG_CHAT}" \\
//                                       --data-urlencode "text=${ZAP_ALERT}" >/dev/null
//                                 '''
//                             }
//                         }
//                         if (zapStatus == 1) {
//                             error('ZAP blocking findings detected')
//                         }
//                         echo 'ZAP warnings reported; continuing pipeline.'
//                     } else {
//                         error("ZAP scan failed to execute (exit ${zapStatus})")
//                     }
//                 }
//             }
//         }
//     }

//     post {

//         success {

//             echo 'Pipeline completed successfully!'

//             sendTelegram('SUCCESS')

//             sendEmail('SUCCESS')

//         }

//         failure {

//             echo 'Pipeline failed. Check Console Output!'

//             sendTelegram('FAILURE')

//             sendEmail('FAILURE')

//         }

//     }

// }

pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 20, unit: 'MINUTES')
    }

    triggers {
        // Run once every day, Jenkins chooses the minute
        cron('H H * * *')
    }

    environment {
        TARGET_URL = 'https://thatdevops.duckdns.org/'
    }

    stages {
        stage('1. Prepare Scan') {
            steps {
                sh '''
                    set -eu
                    mkdir -p zap-reports
                    rm -f zap-reports/zap-report.json \
                          zap-reports/zap-report.html \
                          zap-reports/telegram-message.txt
                    chmod 777 zap-reports
                '''
            }
        }

        stage('2. OWASP ZAP Scan') {
            steps {
                script {
                    int status = sh(
                        script: '''
                            docker run --rm \
                              -v "$PWD/zap-reports:/zap/wrk:rw" \
                              ghcr.io/zaproxy/zaproxy:stable \
                              zap-baseline.py \
                              -t "$TARGET_URL" \
                              -m 1 \
                              -J zap-report.json \
                              -r zap-report.html \
                              -s
                        ''',
                        returnStatus: true
                    )

                    env.ZAP_EXIT_CODE = status.toString()

                    echo "ZAP exit code: ${status}"
                }
            }
        }

        stage('3. Analyze Security Findings') {
            steps {
                sh '''
                    python3 - <<'PY'
import json
import os
from collections import Counter

report = "zap-reports/zap-report.json"
message_file = "zap-reports/telegram-message.txt"
target = os.environ["TARGET_URL"]
code = int(os.environ["ZAP_EXIT_CODE"])

alerts = []

if os.path.isfile(report):
    with open(report, encoding="utf-8") as f:
        data = json.load(f)

    for site in data.get("site", []):
        for alert in site.get("alerts", []):
            alerts.append(alert)

risk_counts = Counter(
    alert.get("riskdesc", "Unknown").split(" ")[0]
    for alert in alerts
)

lines = [
    "JENKINS SECURITY SCAN",
    "",
    f"Website: {target}",
    f"Project: {os.environ.get('JOB_NAME', 'Unknown')}",
    f"Build: #{os.environ.get('BUILD_NUMBER', '?')}",
    "",
    "SECURITY SUMMARY",
    f"High: {risk_counts['High']}",
    f"Medium: {risk_counts['Medium']}",
    f"Low: {risk_counts['Low']}",
    f"Informational: {risk_counts['Informational']}",
    "",
]

if code == 3 or (code != 0 and not alerts):
    lines.append("ERROR: Scan failed or report unavailable.")
elif alerts:
    lines.append("PROBLEMS DETECTED:")
    for i, alert in enumerate(alerts[:8], start=1):
        name = alert.get("name", "Unknown issue")
        risk = alert.get("riskdesc", "Unknown")
        instances = alert.get("instances", [])
        url = (
            instances[0].get("uri", target)
            if instances else target
        )

        lines.extend([
            f"{i}. {name}",
            f"Risk: {risk}",
            f"Instances: {len(instances)}",
            f"URL: {url[:180]}",
            ""
        ])

    if len(alerts) > 8:
        lines.append(
            f"...and {len(alerts) - 8} more alerts."
        )
else:
    lines.append("No security warnings detected.")

lines.extend([
    "",
    "Full ZAP Report:",
    os.environ.get("BUILD_URL", "") +
    "artifact/zap-reports/zap-report.html"
])

message = "\\n".join(lines)[:3900]

with open(message_file, "w", encoding="utf-8") as f:
    f.write(message)

print("Total alert types:", len(alerts))
print("Risk counts:", dict(risk_counts))
PY
                '''
            }
        }

        stage('4. Send Telegram Alert') {
            steps {
                script {
                    int code = env.ZAP_EXIT_CODE.toInteger()

                    if (code == 0) {
                        echo 'Scan clean. No alert required.'
                    } else {
                        withCredentials([
                            string(
                                credentialsId: 'telegram-bot-token',
                                variable: 'TG_TOKEN'
                            ),
                            string(
                                credentialsId: 'telegram-chat-id',
                                variable: 'TG_CHAT_ID'
                            )
                        ]) {
                            int result = sh(
                                script: '''
                                    set +x
                                    curl --fail --silent \
                                      --show-error \
                                      --retry 2 \
                                      --connect-timeout 10 \
                                      --max-time 30 \
                                      -X POST \
                                      "https://api.telegram.org/bot${TG_TOKEN}/sendMessage" \
                                      --data-urlencode "chat_id=${TG_CHAT_ID}" \
                                      --data-urlencode \
                                      "text@zap-reports/telegram-message.txt" \
                                      > /dev/null
                                ''',
                                returnStatus: true
                            )

                            if (result != 0) {
                                echo "Telegram delivery failed: ${result}"
                            } else {
                                echo 'Telegram alert sent.'
                            }
                        }
                    }
                }
            }
        }

        stage('5. Evaluate Scan Result') {
            steps {
                script {
                    int code = env.ZAP_EXIT_CODE.toInteger()

                    if (code == 1) {
                        error('ZAP reported FAIL findings.')
                    }

                    if (code == 2) {
                        unstable('ZAP detected security warnings.')
                    }

                    if (code == 3 || code > 3) {
                        error("ZAP scan error: ${code}")
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts(
                artifacts: 'zap-reports/zap-report.*',
                allowEmptyArchive: true
            )
        }
    }
}
