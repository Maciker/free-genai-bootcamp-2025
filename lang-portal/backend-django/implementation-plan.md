# Django Backend Implementation Plan

## 1. Project Structure Setup
- Create Django project with recommended structure
- Configure SQLite3 database
- Set up Django REST framework for API endpoints

## 2. Database Models
Implement Django models based on schema:
- Word (kanji, romaji, english, parts)
- Group (name, words_count)
- WordGroup (many-to-many relationship)
- StudyActivity (name, url)
- StudySession (group, study_activity)
- WordReviewItem (word, session, correct)

## 3. API Endpoints Implementation
Implement RESTful endpoints with pagination and sorting:

### GET /words
- Paginated list of words (50 items per page)
- Sort by: kanji, romaji, english, correct_count, wrong_count
- Include review statistics in response
- Response format:
  ```json
  {
    "words": [
      {
        "id": number,
        "kanji": string,
        "romaji": string,
        "english": string,
        "correct_count": number,
        "wrong_count": number
      }
    ],
    "total_pages": number,
    "current_page": number,
    "total_words": number
  }
  ```

### GET /words/:id
- Get single word details with related group information
- Response includes word details and associated groups
- Response format:
  ```json
  {
    "word": {
      "id": number,
      "kanji": string,
      "romaji": string,
      "english": string,
      "correct_count": number,
      "wrong_count": number,
      "groups": [
        {
          "id": number,
          "name": string
        }
      ]
    }
  }
  ```

### GET /groups
- Paginated list of word groups (50 items per page)
- Include word counts from counter cache
- Sort by: name, words_count
- Response format:
  ```json
  {
    "groups": [
      {
        "id": number,
        "name": string,
        "words_count": number
      }
    ],
    "total_pages": number,
    "current_page": number,
    "total_groups": number
  }
  ```

### GET /groups/:id
- Get single group details with word count
- Response format:
  ```json
  {
    "id": number,
    "group_name": string,
    "word_count": number
  }
  ```

### GET /groups/:id/words
- Get paginated list of words in group (10 items per page)
- Sort by: kanji, romaji, english, correct_count, wrong_count
- Response format matches GET /words endpoint

### GET /groups/:id/words/raw
- Get unpaginated list of all words in group with full details
- Includes word parts information
- Response format:
  ```json
  {
    "group_id": number,
    "group_name": string,
    "words": [
      {
        "id": number,
        "kanji": string,
        "romaji": string,
        "english": string,
        "parts": object
      }
    ]
  }
  ```

### GET /groups/:id/study_sessions
- Get paginated study session history (10 items per page)
- Sort by: startTime, endTime, activityName, groupName, reviewItemsCount
- Default sort: created_at desc (newest first)
- Response format:
  ```json
  {
    "study_sessions": [
      {
        "id": number,
        "group_id": number,
        "group_name": string,
        "study_activity_id": number,
        "activity_name": string,
        "start_time": string,
        "end_time": string,
        "review_items_count": number
      }
    ],
    "total_pages": number,
    "current_page": number
  }
  ```

### POST /study_sessions
- Create new study session
- Required fields:
  ```json
  {
    "group_id": number,
    "study_activity_id": number
  }
  ```
- Validates group and study activity existence
- Returns created session ID with 201 status
- Response format:
  ```json
  {
    "session_id": number
  }
  ```

### POST /study_sessions/:id/review
- Log word review attempt in study session
- Required fields:
  ```json
  {
    "word_id": number,
    "correct": boolean
  }
  ```
- Validates word and session existence
- Creates word_review_items entry for session tracking
- Updates aggregated statistics in word_reviews table
- Updates last_reviewed timestamp
- Response format on success:
  ```json
  {
    "message": "Review logged successfully"
  }
  ```

### GET /study_sessions
- Get paginated list of all study sessions (10 per page)
- Includes review counts and session metadata
- Sorted by created_at DESC (newest first)
- Response format matches GET /groups/:id/study_sessions

### GET /study_sessions/:id
- Get detailed study session with reviewed words
- Includes session metadata and paginated word list
- Shows session-specific correct/wrong counts
- Response format matches documentation above

### POST /study_sessions/reset (Admin)
- Clear all study history
- Handles cascading deletes properly
- Success response:
  ```json
  {
    "message": "Study history cleared successfully"
  }
  ```

## 4. Integration & Testing
- Implement unit tests for models
- Test API endpoints
- Verify pagination and sorting
- Test database constraints and relationships

## 5. Documentation
- API documentation
- Setup instructions
- Testing guide

## Technical Decisions

### Django REST Framework Features to Use
- ModelViewSet for consistent CRUD operations
- Custom PageNumberPagination with 50 items per page
- Custom filtering backends for sorting with validation
- Serializers for nested relationships with proper depth control
- Comprehensive error handling with consistent response format

### CORS Configuration
- Dynamic CORS configuration based on study_activities URLs
- Additional allowance for development environments (localhost:8080)
- Methods: GET, POST, PUT, DELETE, OPTIONS
- Allow headers: Content-Type, Authorization

### Database Optimization
- Use Django's built-in migration system
- Implement proper indexes on frequently queried fields:
  - words: kanji, romaji, english
  - groups: name
  - word_groups: word_id, group_id
- Utilize counter cache for words_count with signals
- Optimize queries with select_related and prefetch_related
- Use database constraints to maintain data integrity

### Error Handling
- Consistent error response format:
  ```json
  {
    "error": "Error description"
  }
  ```
- HTTP status codes:
  - 200: Success
  - 400: Bad Request (invalid parameters)
  - 404: Not Found
  - 500: Server Error
- Proper validation of query parameters
- Exception handling with detailed error messages in development

### Testing Strategy
- Unit tests for models
- Integration tests for APIs
- Test pagination edge cases
- Verify sorting functionality

## Implementation Phases

1. **Setup Phase**
   - Project initialization
   - Database configuration
   - REST framework setup

2. **Models Phase**
   - Create models with relationships
   - Implement migrations
   - Add model methods for statistics

3. **API Phase**
   - Implement viewsets
   - Add pagination
   - Configure sorting
   - Handle relationship data

4. **Testing Phase**
   - Write comprehensive tests
   - Verify functionality
   - Performance testing

5. **Documentation Phase**
   - API documentation
   - Setup guide
   - Usage examples

## Next Steps
1. Initialize Django project
2. Start with models implementation
3. Build API endpoints iteratively
4. Add tests alongside development