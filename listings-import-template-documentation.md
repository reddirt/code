# Listings Import Template

This CSV template contains all the fields needed for importing listings data into the pipeline system.

## Required Fields

### Basic Listing Information
- **listing_name** (Required): The name of the business/listing
- **loc_address1** (Required): Primary address line
- **loc_city** (Required): City name
- **loc_state** (Required): State/province code (e.g., "CA", "NY")
- **loc_country** (Required): Country code (e.g., "US", "CA")

### Location Details
- **loc_address2** (Optional): Secondary address line (suite, unit, etc.)
- **loc_region** (Optional): Region/district name
- **loc_zip** (Optional): Postal/ZIP code
- **loc_lat** (Optional): Latitude coordinate (decimal degrees)
- **loc_lng** (Optional): Longitude coordinate (decimal degrees)

### Contact Information
- **ci_name** (Optional): Contact person name
- **ci_email** (Optional): Email address
- **ci_local_phone** (Optional): Local phone number
- **ci_international_phone** (Optional): International phone number
- **ci_website_url** (Optional): Main website URL
- **ci_facebook_url** (Optional): Facebook page URL
- **ci_instagram_url** (Optional): Instagram profile URL
- **ci_youtube_url** (Optional): YouTube channel URL
- **ci_tiktok_url** (Optional): TikTok profile URL
- **ci_twitter_url** (Optional): Twitter profile URL
- **ci_pinterest_url** (Optional): Pinterest profile URL
- **ci_tripadvisor_url** (Optional): TripAdvisor page URL
- **ci_yelp_url** (Optional): Yelp page URL

### Content Information
- **listing_info_description** (Optional): Detailed description of the listing

### Complex Data Fields (JSON Format)
- **images_images** (Optional): JSON array of image objects
  ```json
  [{"id":1,"listing_info_id":1,"image":"https://example.com/image1.jpg","rank":1}]
  ```

- **dates** (Optional): JSON array of opening date objects
  ```json
  [{"start_date":"2024-01-01","end_date":"2024-12-31","description":"Open year-round"}]
  ```

- **hours** (Optional): JSON array of opening hours objects
  ```json
  [{"day":"Monday","open_time":"09:00","close_time":"17:00"},{"day":"Tuesday","open_time":"09:00","close_time":"17:00"}]
  ```

- **attributes** (Optional): JSON array of listing attributes
  ```json
  [{"name":"WiFi","description":"Free WiFi available"},{"name":"Parking","description":"Free parking"}]
  ```

### Source Tracking
- **source_url** (Optional): Original source URL where this data came from
- **source_identifier** (Optional): Unique identifier from the source system

## Usage Instructions

1. **Download the template**: Use the `listings-import-template.csv` file
2. **Fill in your data**: Replace the sample data with your actual listing information
3. **Required fields**: Make sure to fill in at least the required fields (listing_name, loc_address1, loc_city, loc_state, loc_country)
4. **JSON fields**: For complex data (images, dates, hours, attributes), use proper JSON format
5. **Save as CSV**: Ensure the file is saved with UTF-8 encoding
6. **Upload**: Use the pipeline import configuration to upload this file

## Data Validation

The pipeline will validate:
- Required fields are present
- Email addresses are properly formatted
- URLs are valid
- JSON fields contain valid JSON
- Coordinates are within valid ranges
- Phone numbers are properly formatted

## Field Mapping

The pipeline uses field mapping to match your CSV columns to the internal database fields. You can customize the mapping in the import configuration if your CSV uses different column names.

## Examples

### Simple Listing
```csv
listing_name,loc_address1,loc_city,loc_state,loc_country
"Joe's Pizza","456 Oak Street","Springfield","IL","US"
```

### Complete Listing
```csv
listing_name,listing_info_description,loc_address1,loc_city,loc_state,loc_country,loc_lat,loc_lng,ci_name,ci_email,ci_local_phone,ci_website_url
"Joe's Pizza","Best pizza in town with fresh ingredients","456 Oak Street","Springfield","IL","US","39.7817","-89.6501","Joe Smith","joe@joespizza.com","(217) 555-0123","https://www.joespizza.com"
```
