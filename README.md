# 1NCE SIM management script

This Bash script is designed to provide a simple command-line interface for managing 1NCE IoT SIM cards using the 1NCE API, making it easier to integrate with your workflows. 

It allows you to perform various operations such as listing SIM cards, retrieving details, managing connectivity, updating labels, enabling/disabling cards, and more. The script is designed to be simple to use and faster than using `curl` requests manually, and formats outputs for better readability. However, not all API features are included in the script at the moment.

For additional resources and API documentation, refer to the [1NCE Developer Hub](https://help.1nce.com/dev-hub/reference/api-welcome).

## Features
- **API token creation**: Generate a token for authentication and API use.
- **SIM details**: Retrieve detailed information about one or more SIM cards.
- **SIM usage**: Fetch data usage information for a specific SIM card and period.
- **SIM status, quota, and connectivity**: Fetch status, and connectivity of a specific SIM.
- **Event logs**: Retrieve event logs for a specific SIM card.
- **Connectivity reset**: Reset connectivity for a specific SIM.
- **Label management**: Update labels for one or more SIM card(s).
- **Enable/disable SIMs**: Enable or disable one or more SIM card(s).
- **IMEI Lock/Unlock**: Lock or unlock the IMEI for one or more SIM card(s).
- **Action logging**: Log all script actions in a local file.

## Prerequisites
- **Dependencies**: ensure the following tools are installed:
  - `curl`
  - `jq`
  - `bat` or `batcat`
  - `awk`
- **1NCE API access**: you need a valid 1NCE customer account to use the API.

## Usage

```bash
./1nce_script.sh [--option] [<argument(s)>]
```

### Options
- `-T, --get-token`: Generate an API token (expires after one hour).
- `-l, --get-sims (<pages>)`: List details of all SIM cards; optionally specify how many pages of 100 cards to retrieve.
- `-L, --get-sim <iccid>`: List details of a specific SIM card.
- `-S, --get-status <iccid>`: Fetch the status of a specific SIM card, including connectivity and location details.
- `-U, --get-usage <iccid> (<interval>)`: Get daily usage for a specific SIM card (default: last 14 days). Optionally specify a date range.
- `-Q, --get-quota <iccid>`: Get the remaining data quota for a specific SIM card.
- `-E, --get-events <iccid> (<page>)`: Fetch events for a specific SIM card; optionally specify a page number.
- `-C, --get-connectivity <iccid>`: Fetch connectivity information for a specific SIM card.
- `-R, --reset-connectivity <iccid>`: Trigger a connectivity reset for a specific SIM card (asynchronous operation).
- `-la, --label <iccid(s)> <label(s)>`: Set labels for one or more SIM cards (provide ICCIDs and labels as comma-separated lists).
- `-e, --enable <iccid(s)>`: Enable one or more SIM cards (provide ICCIDs as a comma-separated list).
- `-d, --disable <iccid(s)>`: Disable one or more SIM cards (provide ICCIDs as a comma-separated list).
- `-il, --imei-lock <iccid(s)>`: Lock the IMEI for one or more SIM cards (provide ICCIDs as a comma-separated list).
- `-iu, --imei-unlock <iccid(s)>`: Unlock the IMEI for one or more SIM cards (provide ICCIDs as a comma-separated list).

## Examples

1. **Generate an API Token**:
   ```bash
   ./1nce --get-token
   Enter your username: janedoe
   Enter your password: 
   ```

2. **List details of all SIM cards**:
   ```bash
   ./1nce --get-sims
   Label      ICCID               IMSI             MSISDN           IMEI              IMEI lock  Status   Activation date               IP             Quota  Quota status             SMS quota  SMS quota status
   -----      -----               ----             ------           ----              ---------  ------   ---------------               --             -----  ------------             ---------  ----------------
   test1      123456789012345678  XXXXXXXXXXXXXXX  XXXXXXXXXXXXXXX  XXXXXXXXXXXXXXXX  false      Enabled  2022-11-25T12:56:06.000+0000  12.345.67.8    500    More than 20% available  250        More than 20% available
   test2      987654321098765432  XXXXXXXXXXXXXXX  XXXXXXXXXXXXXXX  XXXXXXXXXXXXXXXX  false      Enabled  2022-11-21T11:22:01.000+0000  23.456.78.9    500    More than 20% available  250        More than 20% available
   ```

3. **Get usage for a Specific SIM card**:
   ```bash
   ./1nce --get-usage 123456789012345678 2025-01-31:
   Fetching usage data for ICCID: 12345689012345678 from 2025-01-31 to 2025-02-14… Done.
   Date        TX (MB)               RX (MB)                SMS
   ----        -------               -------                ---
   2025-02-14  0                     0                      0
   2025-02-13  0.004879              0.000958               0
   2025-02-12  0.014598999999999997  0.0066879999999999995  0
   2025-02-11  0.015136              0.005858               0
   2025-02-10  0.014634              0.005846               0
   2025-02-09  0.01348               0.005782               0
   2025-02-08  0.013551              0.006811               0
   2025-02-07  0.013233              0.005301               0
   2025-02-06  0.016562              0.006626               0
   2025-02-05  0.014041              0.005846               0
   2025-02-04  0.0122                0.005758               0
   2025-02-03  0.009689              0.003112               0
   2025-02-02  0.016218              0.005962               0
   2025-02-01  0.012242              0.004854               0
   2025-01-31  0.016298              0.00696                0
   TOTAL       0.18676199999999998   0.07636199999999999    0
   ```

4. **Update labels for multiple SIM cards**:
   ```bash
   ./1nce --label 123456789012345678,987654321098765432 "Label1,Label2"
   Updating labels for ICCIDs: 123456789012345678,987654321098765432… Labels updated successfully.
	```

5. **Disable or enable SIM card(s)**:
   ```bash
   ./1nce --disable 123456789012345678
   Disabling ICCIDs: 123456789012345678… Deactivation request successfully sent.

   ./1nce --enable 123456789012345678,987654321098765432
   Enabling ICCIDs: 123456789012345678,987654321098765432… Activation request successfully sent.
   ```

## Logging
All actions are logged in the `1nce.log` file, which includes timestamps, actions (including tracking label changes), ICCIDs, and HTTP response codes.

```
2025-02-14 01:10:04 | enable | 987654321098765432 | Activation request successfully sent | 201
2025-02-14 01:10:04 | enable | 123456789012345678 | Activation request successfully sent | 201
2025-02-14 01:10:04 | disable | 123456789012345678 | Deactivation request successfully sent | 201
2025-02-14 01:09:11 | label | 987654321098765432 | Label2 (was: test2) | 201
2025-02-14 01:09:11 | label | 123456789012345678 | Label1 (was: test1) | 201
2025-02-14 01:06:53 | get-usage | 123456789012345678 | Done | 0
2025-02-14 01:02:40 | get-sims | All | Done | 1
2025-02-14 01:02:37 | get-token | NA | Token retrieved successfully | 200
```

## Notes
- **Token expiry**: API tokens expire after one hour. Use `--get-token` to generate a new token when needed.
- **Bulk operations**: For actions that allow bulk operations (i.e., `--label`, `--enable`, `--disable`, `--imei-lock`, `--imei-unlock`), provide ICCIDs as a comma-separated list.
- **Date range**: For usage queries, the date range can optionally be specified in the format `YYYY-MM-DD:YYYY-MM-DD`, where start date or end date are facultative (i.e., providing `YYYY-MM-DD:` will show usage from this date, and `:YYYY-MM-DD` the usage until this date).

## About 1NCE
1NCE is a global IoT connectivity provider offering a flat-rate pricing model for IoT connectivity. This script is not affiliated or endorsed by 1NCE and is provided as is, use it at your own risk.

## License
GNU AGPLv3
