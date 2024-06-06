
### FLowDiagram

```mermaid 

sequenceDiagram
    participant User
    participant System

    loop for each date in date_list
        User->>System: Remove existing contents (shutil.rmtree)
        User->>System: Recreate directory (os.makedirs)
        User->>System: Create zip file (data_csv_date.zip)
        loop for each strike_price in strike_prices
            loop for each option_type in ["CE", "PE"]
                User->>System: get_symbol_info
                System->>System: Create directory (symbol_strike_price_option_type)
                loop for each interval in intervals
                    User->>System: get_candle_data
                    System->>System: Save data to CSV
                    loop for each file in directory
                        User->>System: Add file to zip
        User->>System: Move zip file to Google Drive (move_zip_file_to_drive)

```
