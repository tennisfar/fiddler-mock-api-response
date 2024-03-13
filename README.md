# Fiddler Mock API Response

This script for Fiddler allows you to mock API responses based on specific request criteria. It is particularly useful for testing and development purposes when you need to simulate API responses without actually hitting the live endpoints.

## How to Use

1. **Install Fiddler**: If you haven't already, install [Fiddler](https://www.telerik.com/fiddler), a free web debugging tool.

2. **Configure Fiddler**:
    - In Fiddler, and go to `Tools` > `Options` > `HTTPS`.
    - Ensure the `Capture HTTPS CONNECTs` and `Decrypt HTTPS traffic` options are checked.

3. **Configure the Script**:
    - In Fiddler, go to `Rules` > `Customize Rules...` to open the Fiddler Script Editor.
    - In Fiddler Script Editor, go to `Go` > `to OnBeforeResponse`.
    - Inside the `OnBeforeResponse` function, add the `CheckAndMockResponse` lines and `ConvertPathToFilePath` functions (see below).
    - Ensure the file paths in the `ConvertPathToFilePath` function reflect the structure and location of your mock JSON files.

4. **Create Mock JSON Files**:
    - Create JSON files corresponding to the API responses you want to mock.
    - Save these files in the directory specified in the `ConvertPathToFilePath` function.
    - Note: The script replaces forward slashes (`/`) with backslashes (`\`) and question marks (`?`) with underscores (`_`) in the file paths.

5. **Run Fiddler**:
    - Open `https://web.develop.danskespil.dk/` in your browser to ensure Fiddler is capturing traffic.
    - Fiddler should open a "Certificate Error" modal. You need to click "Yes" to trust the Fiddler root certificate, and ignore the errors.
    - With Fiddler running, it will intercept requests and, where matching criteria are met, respond with the contents of the specified mock files.

7. **Test Your Application**:
    - Run your application or make requests to the API endpoints you're mocking. Fiddler will serve the mock responses you've set up.
    - If a mock file is not found, Fiddler will log a message to the console with the required file path.

## Script Functions

Add the following functions to your Fiddler script:

```csharp
static function OnBeforeResponse(oSession: Session) {
    CheckAndMockResponse(oSession, "web.", "/dlo/scapi/danskespil/numbergames/groupplay/groups?includePlayers");
    CheckAndMockResponse(oSession, "web.", "/dli/scapi/danskespil/playeraccountmanagement/playergames/1/365/1/aktive/forhandler");
}

// Checks if the session matches the specified criteria and mocks the response if it does.
static function CheckAndMockResponse(oSession: Session, hostnamePrefix: String, pathAndQueryPrefix: String) {
    if (oSession.hostname.StartsWith(hostnamePrefix) && oSession.PathAndQuery.StartsWith(pathAndQueryPrefix) && oSession.oResponse.headers.ExistsAndContains("Content-Type", "application/json")) {
        var filePath = ConvertPathToFilePath(pathAndQueryPrefix);
        if (System.IO.File.Exists(filePath)) {
            oSession.responseCode = 200;
            var fileContents = System.IO.File.ReadAllText(filePath);
            oSession.utilSetResponseBody(fileContents);
        } else {
            FiddlerObject.log("\n\n    FILE NOT FOUND: " + filePath + "\n");
        }
    }
}

// Converts a given API path to a local file path by replacing slashes with backslashes, 
// replacing invalid filename characters, and appending '.json'.
static function ConvertPathToFilePath(path: String) : String {
    var basePath = "C:\\Projects\\fiddler-mock-api-response";
    var sanitizedPath = path.replace("/", "\\").replace("?", "_");
    var filePath = basePath + sanitizedPath + ".json";
    return filePath;
}
