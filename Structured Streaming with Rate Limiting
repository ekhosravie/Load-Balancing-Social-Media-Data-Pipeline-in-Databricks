from pyspark.sql.functions import from_json, col, window, count

# Replace with your social media API endpoint and schema
endpoint = "https://api.example.com/data"
schema = "name string, content string, timestamp timestamp"

# Create a streaming DataFrame
df = spark \
  .readStream() \
  .format("socket") \
  .option("host", "localhost") \
  .option("port", 9999) \
  .load() \
  .select(from_json(col("value"), schema).alias("data")) \
  .select("data.*") \
  .withWatermark("timestamp", "10 seconds")  # Example watermark for rate limiting by timestamp

# Apply rate limiting based on count per window (adjust as needed)
windowed_count = df.withWatermark("timestamp", "10 seconds") \
  .groupBy(window("timestamp", "1 second")) \
  .count() \
  .filter(col("count") < 100)  # Allow maximum 100 messages per second

# Process the filtered data (replace with your processing logic)
processed_data = windowed_count.select("name", "content")

# Start the streaming query
query = processed_data.writeStream \
  .outputMode("append") \
  .format("delta") \
  .option("checkpointLocation", "/mnt/checkpoint/social_data") \
  .start()

# Wait for termination (optional)
query.awaitTermination()
