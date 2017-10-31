# Notes for openECA-based analytics management

Zhijie Nie

Duotong Yang

Revised on 2017-10-11


## Software Requirement
* openECA v.1.0.5.0+
* openHistorian2
* Visual Studio Community Edition 2015+
* Python 2.7
* Pycharm Community Edition
* MySQL Workbench Community Edition (Including sub-requirement like Visual C/C++ compilers)
* MySQL installer (inside MySQL installer to install MySQL Community Server 5.7.20 and MySQL.NET connector)
* Grafana
* Wix Tools
* NSSM (`https://nssm.cc/`)
* 

## Useful References
* GPA Source Codes: `http://www.gridprotectionalliance.org/NightlyBuilds/`
* openPDC Documentations: `https://github.com/GridProtectionAlliance/openPDC`

## Need to Knows
1. TestHarness is just a tool for developing an analytic - the actual end product will be an 
**installable Windows service**.

2. A openECA-generated C# project includes the following projects
* %ProjectName%Library
* %ProjectName%Service
* %ProjectName%ServiceConsole
* %ProjectName%Setup
* %ProjectName%TestHarness


## How to create measurements in openECA using SQL script
1. Copy `C:\Program Files\openECA\Server\Database Scripts\MySQL\SampleDataSet.sql` to a directory you have full control permissions. If you had already got the modified SampleDataSet.sql, replace the one under Database `Scripts\MySQL`
  
2. Add measurement as follows:
```sql
INSERT INTO Measurement(HistorianID, DeviceID, PointTag, SignalTypeID, PhasorSourceIndex, SignalReference, Description, Enabled) VALUES(1, 1, 'SS_118:MEASB14VOLTV', 3, NULL, 'SS118-MEASB14VOLTV', 'Shadow System for 118-bus system - Bus 14 Voltage Magnitude MeasB14VoltV', 1);
```

3. SignalTypeID Number Reference:

| SignalTypeID | Name | Acronym | Suffix | Abbreviation | 
| :----------- | :--- | :------ | :----- | :----------- |
| 1 | Current Magnitude | IPHM | PM | I |
| 2 | Current Phase Angle | IPHA | PA | IH |
| 3 | Voltage Magnitude | VPHM | PM | V | 
| 4 | Voltage Phase Angle | VPHA | PA | VH |
| 5 | Frequency | FREQ | FQ | F | 
| 6 | Frequency Delta (dF/dt) | DFDT | DF | DF |
| 7 | Analog Value | ALOG | AV | AV | 
| 8 | Status Flags | FLAG | SF | S | 
| 9 | Digital Value | DIGI | DV | DV | 
| 10 | Calculated Value | CALC | CV | CV | 
| 11 | Statistic | STAT | ST | ST | 
| 12 | Alarm | ALRM | AL | AL | 
| 13 | Quality Flags | QUAL | QF | QF | 

4. Run `C:\Program Files\openECA\Server\ConfigurationSetupUtility.exe` and set up a MySQL database

5. Open openECA Manager to check if the measurements are added as expected

## How to delete auxiliary files under the solution folder
### Log Files for openECA projects running as service
* FileType: *.logz
* Path: `~\ShadowSys118\Deploy\Debug\Service\Logs\`
* These log files are auto-generated each time running **%ProjectName%Service** under the openECA solution. It contains the all the messages shown in the **Output** tab of Visual Studio.
* These log files are viewable by opening with `C:\Program Files\openECA\Server\LogFileViewer.exe`.
* To clear log: Simply delete all the *.logz files under this path.



### Log File for PSS/E Power Flow Calculation Result
* FileType: *.csv
* Path: `~\ShadowSys118\Log\Outputs.csv`
* Generated by `~\ShadowSys118\PythonCode\CalculatePowerFlow.py`
* To clear log: Double-click `~\ShadowSys118\Log\CleanLogFiles.pyw` and open it with `C:\Python27\pythonw.exe`
* To disable log: Comment the following codes in `~\ShadowSys118\PythonCode\CalculatePowerFlow.py`
```python
    # region [ Create a new line in Output.csv file ]
    outputs_csvfile = open(OutputsFilePath, 'a')
    wcsv = csv.writer(outputs_csvfile, delimiter=',', lineterminator='\n')
    wcsv.writerow(_frame)
    outputs_csvfile.close()
    # endregion
```
* This file is generated for data visualization and export in the early stage usage. The power flow data can be better visualized using Grafana Dashboard. And data export can also be accomplished by setting up a new output adapter like **CsvAdapters.CsvOutputAdapter** under openECA Manager. Bulk data exportation using openHistorian is preferred.


## How to display openHistorian measurements in Grafana

### Install Grafana Dashboard
* Installing on Windows: `http://docs.grafana.org/installation/windows/`
* Grafana default port: `localhost:3000`
* Grafana Admin Username: `admin`
* Grafana Admin Password: `admin`


### Create Internal Data Connections from openECA to openHistorian
1. Run openHistorian Manager, go to the tab ***Inputs >> Subscription Based Inputs >> Create Internal Subscription***
2. Set up the **Acronym** and **Name** as you like
3. Type `localhost` in **Hostname** and set the port number as `6190` (**dataPublisherPort** for openECA data usage)
4. Check **Receive Internal Metadata**
5. Uncheck **Receive External Metadata**
6. Click **Next** button
7. Click **Save** and **Initialize**
8. Go to the tab ***Metadata >> Measurements*** to check whether the internal data connection is sucessful or not; the subcribed measurements are added at the end of the list with new ID assigned by openHistorian.
9. Go to the tab ***Monitor >> Graph Measurements*** to see if the subcribed measurements can be displayed on the graph window
10. The zip file contains a folder with the current Grafana version. Extract this folder to anywhere you want Grafana to run from. Go into the `conf` directory and make a copy of `sample.ini` as `custom.ini`. You should edit `custom.ini`, never `defaults.ini`.
11. The default Grafana port is `3000`, this port requires extra permissions on windows. Edit custom.ini and uncomment the `http_port` configuration option (; is the comment character in ini files) and change it to something like `http_port = 8080` or similar. That port should not require extra Windows privileges.
12. Start Grafana by executing `grafana-server.exe` under the directory `\usr\grafana\bin`, preferably from the command line. If you want to run Grafana as windows service, download `NSSM`. It is very easy add Grafana as a Windows service using that tool.

The following link is the similar reference for creating internal gateway connections in **SIEGate**:
`https://github.com/GridProtectionAlliance/SIEGate/blob/master/Source/Documentation/wiki/Creating_Internal_Gateway_Connections.md`

### Add openHistorian as a plug-in to Grafana
1. Run command prompt window (cmd)

2. Change current directory to Grafana's bin folder:
```
cd C:\Program Files\grafana-4.3.1\bin
```
3. Use the grafana-cli tool to install the openHistorian data source from the command line:
```
grafana-cli plugins install gridprotectionalliance-openhistorian-datasource
```


#### References
* `https://github.com/GridProtectionAlliance/openHistorian-grafana`
* `https://grafana.com/plugins/gridprotectionalliance-openhistorian-datasource`
* `https://grafana.com/plugins/gridprotectionalliance-osisoftpi-datasource`


### Connect openHistorian Data Source in Grafana
1. Open browser and go to `localhost:8080`(It is not '3000' any more since you change 'custom.ini' to 8080)
2. Go to the tab ***Grafana Logo >> Data Sources >> Add data source***.
3. Type `openHistorian Data Source` (as you like) in **Name**, and change the type to `openHistorian` in the drop-down list
4. Type `http://localhost:8180/api/grafana/` in **Url**.
5. Click **Add** or **Save & Test**
6. In a new dashboard, click ***Panel Title >> Edit >> Metrics***, and you will find the openHistorian measurement (including internal subscription from openECA) under the drop-down list of **select metric**


## How to release an installble file (*.msi) to deploy for other PCs
This topic is related to Wix tools to generate a Microsoft Installer (*.msi) file. Install Wix tools properly first. Make sure the icons under `%ProjectName%Setup` are colorful and displayed in Visual Studio. Then ask Kevin about the accurate procedures for creating an installer of analytics.


## How to deploy Shadow System Simulator as a Windows Service
1. Install NSSM, see 
2. Run cmd and change directory as follows:
```
cd C:\Program Files\nssm
nssm install
```
3. Select the path as `~\ShadowSys\Deploy\Debug\Service\ShadowSysService.exe`
4. Fill in the service name as you like and click **Install**
5. Run `services.msc`
6. Start the service that you have deployed


