# CDE_Linux_GIT

# Extract: Download the CSV file and save it into the raw folder
echo "Starting extraction process..."
mkdir -p "$RAW_DIR"
curl -s "$CSV_URL" -o "$RAW_DIR/raw_file.csv"

if [ -f "$RAW_DIR/raw_file.csv" ]; then
  echo "File successfully downloaded and saved to $RAW_DIR."
else
  echo "File download failed."
  exit 1
fi

# Transform: Rename the column and select specific columns
echo "Starting transformation process..."
mkdir -p "$TRANSFORMED_DIR"

awk -F',' 'NR==1 {for (i=1; i<=NF; i++) if ($i=="Variable_code") $i="variable_code"} NR==1 || ($1!="year" && $3!="Value" && $4!="Units" && $5!="variable_code") {print $1","$2","$3","$4}' "$RAW_DIR/raw_file.csv" > "$TRANSFORMED_DIR/$TRANSFORMED_FILE"

if [ -f "$TRANSFORMED_DIR/$TRANSFORMED_FILE" ]; then
  echo "Transformation completed and file saved to $TRANSFORMED_DIR."
else
  echo "Transformation failed."
  exit 1
fi

# Load: Move the transformed file to the gold directory
echo "Starting load process..."
mkdir -p "$GOLD_DIR"
mv "$TRANSFORMED_DIR/$TRANSFORMED_FILE" "$GOLD_DIR/"

if [ -f "$GOLD_DIR/$TRANSFORMED_FILE" ]; then
  echo "File successfully loaded into $GOLD_DIR."
else
  echo "Load process failed."
  exit 1
fi

echo "ETL process completed successfully."

####Comment: Schedule the script using crontab
#### crontab -e to open the cron tab and then type the line below for 12:00AM daily
#### 0 0 * * * ~/Desktop/CDE/CDE_Linux_Git_Assignment/scripts.sh
