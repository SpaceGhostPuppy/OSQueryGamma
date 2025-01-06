# Installieren unter macOS

Continuous Integration testet derzeit macOS Builds von osquery gegen macOS 11 (siehe die os: [macos- Zeile in der build_macos Sektion der CI-Konfiguration. Die gesamte Kernfunktionalität von osquery sollte auf macOS 10.15 oder neuer funktionieren, obwohl einige Tabellen Daten lesen können, die nur auf bestimmten Versionen von macOS vorhanden sind, da Apple neue Datenquellen hinzufügt oder andere veraltet sind. Versionen von macOS 10.14 und älter werden nicht mehr unterstützt.

# Paket-Installation

Wenn Sie einen unternehmensweiten Einsatz von osquery planen, ist die einfachste Installationsmethode ein macOS-Paketinstallationsprogramm. Sie müssen Updates verwalten und verteilen.

Jeder osquery-Tag (Release) erstellt ein macOS-Paket: osquery.io/downloads. Es gibt keine Abhängigkeiten von Paketen oder Bibliotheken.

Das Standardpaket erstellt die folgende Struktur:

/private/var/osquery/io.osquery.agent.plist
/private/var/osquery/osquery.example.conf
/private/var/log/osquery/
/private/var/osquery/lenses/{*}.aug
/private/var/osquery/packs/{*}.conf
/opt/osquery/lib/osquery.app
/usr/local/bin/osqueryi -> /opt/osquery/lib/osquery.app/Contents/MacOS/osqueryd
/usr/local/bin/osqueryctl -> /opt/osquery/lib/osquery.app/Contents/Resources/osqueryctl

Dieses Paket installiert keinen LaunchDaemon zum Starten von osqueryd. Sie können das Skript osqueryctl start verwenden, um die Beispiel-LaunchDaemon-Job-Pliste und die zugehörige Konfiguration zu kopieren.


# Hinweis zum Upgrade von osquery 4.x auf 5.x

Beim Upgrade von älteren Versionen auf neuere bietet osquery selbst keinen Mechanismus, um den Dienst der älteren Version zu stoppen, osquery zu aktualisieren und dann den Dienst neu zu starten.

# Schritte nach der Installation

Diese Schritte gelten nur, wenn dies das erste Mal ist, dass Sie osqueryd auf diesem Mac installieren und ausführen.

Führen Sie nach Abschluss der Paketinstallation die folgenden Befehle aus. Wenn Sie das Rezept zur Installation von osquery verwenden, sind diese Schritte nicht notwendig: Das Rezept deckt dies ab. https://osquery.readthedocs.io/en/latest/deployment/configuration/#chef-macos

_You can use the helper script:_
sudo osqueryctl start

_Or, install the example config and launch daemon yourself:_
sudo cp /var/osquery/osquery.example.conf /var/osquery/osquery.conf
sudo cp /var/osquery/io.osquery.agent.plist /Library/LaunchDaemons
sudo launchctl load /Library/LaunchDaemons/io.osquery.agent.plist

# Entfernen von osquery

Um osquery von einem macOS-System zu entfernen, führen Sie die folgenden Befehle aus:

_Unload and remove io.osquery.agent.plist launchdaemon_
sudo launchctl unload /Library/LaunchDaemons/io.osquery.agent.plist
sudo rm /Library/LaunchDaemons/io.osquery.agent.plist

_Remove files/directories created by osquery installer pkg_
sudo rm -rf /private/var/log/osquery
sudo rm -rf /private/var/osquery
sudo rm /usr/local/bin/osquery*
sudo rm -rf /opt/osquery

sudo pkgutil --forget io.osquery.agent


Um eine eigenständige osquery zu starten, verwenden Sie: osqueryi. Dies benötigt keinen Server oder Dienst. Alle Tabellenimplementierungen sind enthalten!

Nachdem Sie den Rest der Dokumentation durchgelesen haben, sollten Sie die Grundlagen der Konfiguration und der Protokollierung verstehen. Diese und die meisten anderen Konzepte gelten für osqueryd, den Daemon.



