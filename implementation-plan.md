# Implementation Plan: POST /study_sessions Route

## Overview
This endpoint will create a new study session for a group with a specific study activity.

## Prerequisites
- [ ] Flask development environment is set up
- [ ] Database connection is working
- [ ] Understanding of the existing study_sessions table structure

## Implementation Steps

### 1. Basic Route Setup
- [ ] Add new route decorator for POST /api/study-sessions
- [ ] Add cross_origin decorator
- [ ] Create basic function structure

Test checkpoint:
```python
@app.route('/api/study-sessions', methods=['POST'])
@cross_origin()
def create_study_session():
    return jsonify({"message": "Route working"}), 200
```

### 2. Request Validation
- [ ] Add validation for required fields in request body:
  - group_id
  - study_activity_id
- [ ] Validate data types (integers)

Test checkpoint:
```python
# Test with curl or Postman:
curl -X POST http://localhost:5000/api/study-sessions \
  -H "Content-Type: application/json" \
  -d '{"group_id": 1, "study_activity_id": 1}'

# Expected validation code:
def create_study_session():
    data = request.get_json()
    
    if not data:
        return jsonify({"error": "No data provided"}), 400
        
    group_id = data.get('group_id')
    study_activity_id = data.get('study_activity_id')
    
    if not group_id or not study_activity_id:
        return jsonify({"error": "Missing required fields"}), 400
```

### 3. Database Validation
- [ ] Check if group exists
- [ ] Check if study activity exists
- [ ] Return appropriate error if either doesn't exist

Test checkpoint:
```python
# Add these queries to validate existence:
cursor = app.db.cursor()

cursor.execute('SELECT id FROM groups WHERE id = ?', (group_id,))
if not cursor.fetchone():
    return jsonify({"error": "Group not found"}), 404

cursor.execute('SELECT id FROM study_activities WHERE id = ?', (study_activity_id,))
if not cursor.fetchone():
    return jsonify({"error": "Study activity not found"}), 404
```

### 4. Create Study Session
- [ ] Insert new record into study_sessions table
- [ ] Get the created session ID
- [ ] Return the created session data

Test checkpoint:
```python
# Insert query:
cursor.execute('''
    INSERT INTO study_sessions (group_id, study_activity_id, created_at)
    VALUES (?, ?, ?)
''', (group_id, study_activity_id, datetime.utcnow()))

session_id = cursor.lastrowid
app.db.commit()
```

### 5. Return Session Details
- [ ] Query full session details including related data
- [ ] Format response similar to GET endpoint
- [ ] Return with 201 status code

Test checkpoint:
```python
# Query new session details:
cursor.execute('''
    SELECT 
        ss.id,
        ss.group_id,
        g.name as group_name,
        sa.id as activity_id,
        sa.name as activity_name,
        ss.created_at
    FROM study_sessions ss
    JOIN groups g ON g.id = ss.group_id
    JOIN study_activities sa ON sa.id = ss.study_activity_id
    WHERE ss.id = ?
''', (session_id,))

session = cursor.fetchone()
return jsonify({
    'id': session['id'],
    'group_id': session['group_id'],
    'group_name': session['group_name'],
    'activity_id': session['activity_id'],
    'activity_name': session['activity_name'],
    'start_time': session['created_at'],
    'end_time': session['created_at'],
    'review_items_count': 0  # New session starts with 0 reviews
}), 201
```

## Testing Plan

### Manual Testing
1. Test successful creation:
```bash
curl -X POST http://localhost:5000/api/study-sessions \
  -H "Content-Type: application/json" \
  -d '{"group_id": 1, "study_activity_id": 1}'
```

2. Test validation errors:
```bash
# Missing fields
curl -X POST http://localhost:5000/api/study-sessions \
  -H "Content-Type: application/json" \
  -d '{}'

# Invalid group_id
curl -X POST http://localhost:5000/api/study-sessions \
  -H "Content-Type: application/json" \
  -d '{"group_id": 999999, "study_activity_id": 1}'
```

### Automated Testing (Python/pytest)
```python
def test_create_study_session(client):
    # Test successful creation
    response = client.post('/api/study-sessions', 
                         json={'group_id': 1, 'study_activity_id': 1})
    assert response.status_code == 201
    assert 'id' in response.json
    
    # Test missing data
    response = client.post('/api/study-sessions', json={})
    assert response.status_code == 400
    
    # Test invalid group
    response = client.post('/api/study-sessions', 
                         json={'group_id': 99999, 'study_activity_id': 1})
    assert response.status_code == 404
```

## Success Criteria
- [ ] Endpoint creates new study session successfully
- [ ] All validation checks are working
- [ ] Response format matches GET endpoint
- [ ] Error handling is comprehensive
- [ ] Tests are passing

## Notes
- Remember to handle database transactions properly
- Use appropriate HTTP status codes
- Follow existing code patterns for consistency
- Add appropriate error logging 