# RepairClubAndroidSDK

**RepairClubAndroidSDK** is a Kotlin-based library (AAR) that provides an easy-to-use interface (`RepairClubManager`) for discovering, connecting to, and interacting with Repair Club built devices over BLE on Android. It focuses purely on BLE communication and vehicle data handling without any UI components.

---

## Installation

### 1. Add JitPack Repository

Modify your project’s `settings.gradle.kts` (or `settings.gradle`) to include JitPack:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven("https://jitpack.io")
    }
}
```

### 2. Add the SDK Dependency

#### If You Are Using Version Catalog (`libs.versions.toml`)

Add the following entries to your `libs.versions.toml`:

```toml
[versions]
repairclub-sdk = "1.4.63"

[libraries]
# Key Format: group:artifact
repairclub-sdk = { module = "com.github.repairclub:repairclub-android-sdk", version.ref = "repairclub-sdk" }
```

Then, in your app module’s `build.gradle.kts`:

```kotlin
dependencies {
    implementation(libs.repairclubandroidsdk)
    // Other dependencies...
}
```

#### If Not Using Version Catalog

Add directly to your app module’s `build.gradle.kts` (or `build.gradle`):

```kotlin
dependencies {
    implementation("com.github.repairclub:repairclub-android-sdk:1.4.63")
    // Other dependencies...
}
```

### 3. Sync Gradle

Sync your project to fetch and resolve the SDK dependency.

---

## Quick Start

Below is a basic example demonstrating common SDK usage. Place this code in an Activity or ViewModel:

```kotlin
import android.os.Bundle
import android.util.Log
import androidx.appcompat.app.AppCompatActivity
import com.repairclub.sdk.RepairClubManager
import com.repairclub.sdk.models.ConnectionState
import com.repairclub.sdk.models.ScanProgressUpdate
import com.repairclub.sdk.models.DeviceItem
import com.repairclub.sdk.models.ScanEntry
import com.repairclub.sdk.models.ModuleItem

class MainActivity : AppCompatActivity() {
    private val manager: RepairClubManager = RepairClubManager.getInstance(this)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 1. Initialize SDK (already done via getInstance)
        // 2. Configure SDK (e.g., at app launch, after obtaining auth token)
        manager.configureSDK(
            tokenString = "YOUR_AUTH_TOKEN",
            appName = "MyAndroidApp",
            appVersion = "1.0.0",
            userID = "user@example.com"
        )

        // 3. Scan for devices
        manager.returnDevices { devices ->
            Log.d("MainActivity", "Found devices: $devices")
            if (devices.isNotEmpty()) {
                // Example: connect to the first device
                val firstDevice = devices[0]
                connectToDevice(firstDevice)
            }
        }
    }

    private fun connectToDevice(deviceItem: DeviceItem) {
        manager.connectTo(deviceItem) { connectionEntry, stage, state ->
            Log.d("MainActivity", "Connection stage: \$stage, state: \$state")
            if (state == ConnectionState.connected) {
                // 4. Subscribe to VIN updates
                manager.registerVIN { vin, error ->
                    if (vin != null) {
                        Log.d("MainActivity", "Received VIN: \$vin")
                    } else {
                        Log.e("MainActivity", "VIN error: \$error")
                    }
                }

                // 5. Subscribe to vehicle data
                manager.registerVehicle { vehicleEntry, error ->
                    if (vehicleEntry != null) {
                        Log.d("MainActivity", "Vehicle details: \$vehicleEntry")
                    } else {
                        Log.e("MainActivity", "Vehicle error: \$error")
                    }
                }

                // 6. Start a DTC scan
                manager.startTroubleCodeScan(advancedScan = false) { progressUpdate ->
                    when (progressUpdate) {
                        is ScanProgressUpdate.ScanStarted -> {
                            Log.d("MainActivity", "Scan started: \${progressUpdate.scanEntry}")
                        }
                        is ScanProgressUpdate.ProgressUpdate -> {
                            Log.d("MainActivity", "Scan progress: \${progressUpdate.progress}")
                        }
                        is ScanProgressUpdate.ModulesUpdate -> {
                            Log.d("MainActivity", "Modules updated: \${progressUpdate.modules}")
                        }
                        is ScanProgressUpdate.ScanSucceeded -> {
                            Log.d("MainActivity", "Scan succeeded: \${progressUpdate.modules}")
                        }
                        is ScanProgressUpdate.ScanFailed -> {
                            Log.e("MainActivity", "Scan failed")
                        }
                        else -> { /* Other cases */ }
                    }
                }
            }
        }
    }
}
```

---

## Usage Overview

Below are categorized lists of the most commonly used methods exposed by **RepairClubManager**. Refer to the source code for additional details and less frequently used helper methods.

### 1. Initialization & Configuration

- `RepairClubManager.getInstance(context: Context)`  
  Returns the singleton instance and initializes internal components. Must be called before other methods.

- `configureSDK(tokenString: String, appName: String, appVersion: String, userID: String)`  
  Configure the SDK with your authentication token, app name/version, and user identifier.

- `configureSdkUser(userID: String)`  
  Update or switch the user ID at runtime.

---

### 2. Device Discovery & Connection

- `returnDevices(completionHandler: (List<DeviceItem>) -> Unit)`  
  Start BLE scanning and receive discovered `DeviceItem` objects. The callback is invoked initially with an empty list, then repeatedly as devices are found.

- `stopScanning()`  
  Stop an ongoing BLE scan.

- `connectTo(deviceItem: DeviceItem, connectionHandler: (ScanEntry, ConnectionStage, ConnectionState) -> Unit)`  
  Connect to a selected device. Connection updates (entry, stage, and state) are passed to your handler.

- `disconnectFromDevice()`  
  Disconnect from the currently connected device.

- `subscribeToDisconnections(callback: () -> Unit)`  
  Register a callback that is invoked when the BLE connection is lost.

- `getPeripheral(macAddress: String): BluetoothPeripheral?`  
  Retrieve a `BluetoothPeripheral` by its MAC address, if previously cached by the BLE manager.

---

### 3. VIN & Vehicle Information

- `registerVIN(completionHandler: (String?, CommonError?) -> Unit)`  
  Subscribe to asynchronous VIN updates as they become available.

- `registerVehicle(completionHandler: (VehicleEntry?, CommonError?) -> Unit)`  
  Subscribe to asynchronous `VehicleEntry` updates once vehicle details arrive.

- `requestVehicleInformationFor(vinString: String?, completion: (Result<VehicleEntry?>) -> Unit)`  
  Manually request vehicle information by providing a VIN string.

- `supplyVehicleInformation(vin: String?, vehicleEntry: VehicleEntry?)`  
  Supply manual VIN or `VehicleEntry` data when prompted by the device.

- `getCurrentVehicleEntry(): VehicleEntry?`  
  Retrieve the last received or manually supplied `VehicleEntry`, if available.

- `setCurrentVehicleToA(vehicle: VehicleEntry)`  
  Force-set a manual `VehicleEntry` (useful for simulation or when the device asks for manual data).

---

### 4. Diagnostic Trouble Code (DTC) Scans

- `getCurrentScanEntry(scanID: String): Result<ScanEntry>`  
  Fetch an existing `ScanEntry` by its ID, if a scan is in progress or recently completed.

- `startTroubleCodeScan(advancedScan: Boolean = false, progressHandler: (ScanProgressUpdate) -> Unit)`  
  Start a generic or advanced DTC scan. Progress updates are provided via the `ScanProgressUpdate` sealed class (scan started, progress percentage, module updates, scan success/failure).

- `startExtendedTroubleCodeScan(progressHandler: (ScanProgressUpdate) -> Unit)`  
  Start an extended scan that requests additional data.

- `stopTroubleCodeScan()`  
  Stop an in-progress DTC scan immediately.

- `subscribeToAdvancedDataScanEntries(completionHandler: ((List<ModuleItem>, CommonError) -> Unit)?)`  
  Subscribe to asynchronous module entry updates (`ModuleItem` objects) during an advanced scan.

---

### 5. Live Data (Mode 01) & Monitors

- `subscribeToLiveStream(completionHandler: (Result<Map<String, LiveFeedDatedEntry>>) -> Unit)`  
  Subscribe to live data (Mode 01) feed updates.

- `subscribeToLiveDataRanges(completionHandler: (Result<Pair<Map<String, Double>, Map<String, Double>>>) -> Unit)`  
  Subscribe to live data range updates (min/max values for supported PIDs).

- `requestAllLiveDataSources(): Result<List<LiveDataSource>>`  
  Retrieve a list of every possible Mode 01 PID supported by the vehicle.

- `requestAvailableLiveDataSources(callback: (ArrayList<LiveDataSource>?, CommonError?) -> Unit)`  
  Retrieve currently available PIDs that the vehicle reports.

- `startLiveDataStream(codes: List<LiveDataSource>): Result<Boolean>`  
  Begin streaming live data for the selected PIDs (max 8 at a time).

- `endLiveDataStream(): Boolean`  
  Stop the current live data stream.

- `currentLiveFeedCSV(countToKeep: Int = 1000): Result<String>`  
  Get a CSV-formatted snapshot of the current live data buffer.

- `returnLiveFeedCSV(feedEntries: List<LiveFeedDatedEntry>, countToKeep: Int = 1000): Result<String>`  
  Build a CSV string for a supplied list of live data entries.

---

#### Mode 06 & Readiness Monitors

- `subscribeToMonitors(requestType: Int = 0, completionHandler: (Result<List<ValueMonitor>>) -> Unit)`  
  Subscribe to Mode 06 readiness monitors (one-shot or continuous looping). Returns a list of `ValueMonitor` objects.

- `subscribeToMode06Monitors(completionHandler: (Result<List<String>>) -> Unit)`  
  Retrieve supported Mode 06 PIDs (list of string MID codes).

- `requestMonitors(requestType: Int = 0, sparkVehicle: Boolean = true)`  
  Send a direct readiness request with optional vehicle spark parameter.

- `startRequestMonitorsTimer(requestType: Int = 0)`  
  Begin continuous readiness looping with built-in timer.

- `endRequestMonitorsTimer()`  
  Stop readiness looping timer.

- `registerMilChange(completionHandler: (Boolean?, CommonError?) -> Unit)`  
  Subscribe to MIL (Malfunction Indicator Lamp) state changes (boolean value).

- `requestMilStatus(): Result<Boolean>`  
  Get the current MIL status (immediate result if available, or failure if not).

---

### 6. Freeze Frame

- `requestFreezeFrameFor(code: TroubleCodeEntry, scanEntry: ScanEntry, completionHandler: (Result<List<ValueMonitor>>) -> Unit)`  
  Request full freeze-frame monitors for a given trouble code.

- `requestFreezeFrameValuesFor(code: TroubleCodeEntry, scanEntry: ScanEntry, valueCompletionHandler: (Result<Map<String, Double>>) -> Unit)`  
  Request only freeze-frame PID values (raw map of PID to value).

---

### 7. VIN Decoding & Vehicle APIs

- `requestVinDecode(vin: String, completionHandler: (Result<VINResult>) -> Unit)`  
  Decode a VIN string via network, returning a `VINResult`.

- `requestVinDetailDecode(vin: String, completionHandler: (Result<VINDetailResult>) -> Unit)`  
  Get detailed VIN decode info (manufacturer, engine, etc.).

- `requestVehicleID(for: vehicleEntry: VehicleEntry, completionHandler: ...)`  
  *[If implemented]* Request NHTSA vehicle ID by vehicle model/year.

- `requestSafetyRatingsFor(id: String, completionHandler: (Result<VehicleSafetyResults>) -> Unit)`  
  Fetch NHTSA safety ratings by vehicle ID.

- `requestMakeList(completion: (Result<MakeResults?>) -> Unit)`  
  Fetch a list of vehicle makes (useful for building UI pickers).

- `requestModelsFor(makeID: Int, completion: (Result<ModelResults?>) -> Unit)`  
  Fetch vehicle models for a given make ID.

---

### 8. Clearing Codes & Advanced Services

- `clearGenericCodes(completionHandler: (Result<Boolean>) -> Unit)`  
  Clear generic (Mode 03) DTCs. On success, triggers a readiness monitor request after a 2-second delay.

- `clearAllCodes(progressHandler: (ScanProgressUpdate) -> Unit)`  
  Clear all codes (generic + advanced) with progress updates returned via `ScanProgressUpdate`.

- `subscribeAvailableServices(completion: (List<AdvancedService>?) -> Unit)`  
  Subscribe to a callback providing available advanced service types (e.g., OEM-specific cal/programming features).

- `fetchAvailableServices(): List<AdvancedService>`  
  Synchronous fetch of available advanced services (throws if session is missing).

- `startAdvancedService(service: AdvancedService, progressHandler: (ScanProgressUpdate) -> Unit)`  
  Initiate a given advanced service with progress updates.

---

### 9. Firmware Updates

- `subscribeToFirmwareProgress(completionHandler: (FirmwareProgress) -> Unit)`  
  Subscribe to ongoing firmware update progress (`FirmwareProgress` contains percentage and status).

- `cancelDeviceFirmwareUpdate(completionCallback: (Result<Boolean>) -> Unit)`  
  Cancel an in-progress firmware update.

- `startDeviceFirmwareUpdate(reqVersion: String? = null, reqReleaseLevel: FirmwareReleaseType? = null, progressCallback: (Double) -> Unit, completionCallback: (Result<Boolean>) -> Unit)`  
  Begin a versioned firmware update. `reqReleaseLevel` can be PRODUCTION, BETA, etc.

- `stopDeviceFirmwareUpdate()`  
  Stop any in-progress firmware update and disconnect from the device.

- `subscribeToFirmwareVersionChanges(completionCallback: (String, String) -> Unit)`  
  Subscribe to firmware version changes (current vs. newest available).

- `getDeviceFirmwareVersion(): Result<String?>`  
  Get the current connected device firmware version (or failure if not connected).

- `getNewestAvailableFirmwareVersion(reqVersion: String? = null, reqReleaseLevel: FirmwareReleaseType = FirmwareReleaseType.PRODUCTION, refresh: Boolean = false, callback: (Result<String?>) -> Unit)`  
  Fetch the newest available firmware version (with optional refresh).

---

### 10. Utilities & Cache Management

- `clearCache(cacheType: CacheType): Result<Boolean>`  
  Clear specific SDK caches:  
  - `ENCRYPTION`: Encryption session data  
  - `VEHICLES`: Stored vehicle information  
  - `CONFIG`: Stored vehicle configurations  
  - `CLEAR_ALL`: All implemented caches

- `subscribeToVehicleInfoErrors(completionCallback: (VehicleInfoRequest.Reason) -> Unit)`  
  Subscribe to vehicle info request errors (callback with `VehicleInfoRequest.Reason`).

---

## Example Flow

Below is a step-by-step example of a typical flow in an Android application:

1. **App Launch**  
   - In your `Application.onCreate()`, call:  
     ```kotlin
     class MyApp : Application() {
         override fun onCreate() {
             super.onCreate()
             RepairClubManager.getInstance(this)
         }
     }
     ```

2. **User Authentication & SDK Configuration**  
   - After the user logs in or obtains an auth token, call:
     ```kotlin
     RepairClubManager.getInstance().configureSDK(
         tokenString = userToken,
         appName = "MyApp",
         appVersion = BuildConfig.VERSION_NAME,
         userID = currentUser.id
     )
     ```

3. **Scan for Devices**  
   ```kotlin
   RepairClubManager.getInstance().returnDevices { deviceList ->
       if (deviceList.isNotEmpty()) {
           val device = deviceList[0]
           connectToDevice(device)
       }
   }
   ```

4. **Connect to Device**  
   ```kotlin
   private fun connectToDevice(deviceItem: DeviceItem) {
       RepairClubManager.getInstance().connectTo(deviceItem) { entry, stage, state ->
           if (state == ConnectionState.connected) {
               // Now you can request VIN, vehicle info, etc.
               subscribeToVIN()
               subscribeToVehicleData()
               startDTCScan()
           }
       }
   }
   ```

5. **Subscribe to VIN & Vehicle Data**  
   ```kotlin
   private fun subscribeToVIN() {
       RepairClubManager.getInstance().registerVIN { vin, error ->
           if (vin != null) {
               Log.d("App", "VIN: $vin")
           }
       }
   }

   private fun subscribeToVehicleData() {
       RepairClubManager.getInstance().registerVehicle { vehicleEntry, error ->
           if (vehicleEntry != null) {
               Log.d("App", "Vehicle: $vehicleEntry")
           }
       }
   }
   ```

6. **Perform a DTC Scan**  
   ```kotlin
   private fun startDTCScan() {
       RepairClubManager.getInstance().startTroubleCodeScan(advancedScan = false) { update ->
           when (update) {
               is ScanProgressUpdate.ScanStarted -> Log.d("App", "Scan started")
               is ScanProgressUpdate.ProgressUpdate -> Log.d("App", "Progress: ${update.progress}")
               is ScanProgressUpdate.ModulesUpdate -> Log.d("App", "Modules: ${update.modules}")
               is ScanProgressUpdate.ScanSucceeded -> Log.d("App", "Scan success: ${update.modules}")
               is ScanProgressUpdate.ScanFailed -> Log.e("App", "Scan failed")
           }
       }
   }
   ```

---

## Debugging & Logging (Optional)

```kotlin
import android.util.Log

Log.d("RepairClubSDK", "SDK Initialized Successfully")
```

---

## Additional Notes

- Ensure required BLE and location permissions are declared in your `AndroidManifest.xml`.
- If using ProGuard/R8, add necessary keep rules to prevent code stripping:
  ```
  -keep class com.repairclub.sdk.** { *; }
  ```

---

