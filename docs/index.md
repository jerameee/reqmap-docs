# Overview

ReqMap is a Flask-based web application for managing and visualizing requirements. It allows users to create, update, and delete requirements, as well as view a traceability diagram of the requirements.

## Architecture

The application follows a Model-View-Controller (MVC) architecture:

* __Model:__ Defined in ```models.py''''
* __View:__ HTML Templates (```index.html```) and CSS (```styles.css```)
* __Controller:__ Main application logic in```app.py```

## Dependencies

Key dependencies are listed in ```requirements.txt```

``` txt
Flask
Flask-SQLAlchemy
networkx
matplotlib
```

## Database Model

THe core data model is the ```Requirement``` class, defined in ```models.py```

``` py
class Requirement(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(150), nullable=False)
    description = db.Column(db.Text, nullable=False)
    priority = db.Column(db.String(50), nullable=False)
    status = db.Column(db.String(50), nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, onupdate=datetime.utcnow)
    version = db.Column(db.Integer, default=1)
    parent_id = db.Column(db.Integer, db.ForeignKey("requirement.id"), nullable=True)
    parent = db.relationship("Requirement", remote_side=[id])
```

This model includes fields for the requirement's title, description, priority, status, creation and update timestamps, version, and a self-referential relationship for parent requirements.

## Key Functionalities

### Creating a Requirement

Endpoint: `POST/requirements`

``` py
@app.route('/requirements', methods=['POST'])
def create_requirement():
    data = request.get_json()
    new_req = Requirement(
        title=data['title'],
        description=data['description'],
        priority=data['priority'],
        status=data['status']
    )
    db.session.add(new_req)
    db.session.commit()
    return jsonify({'message': 'Requirement created successfully!'}), 201
```

### Updating a Requirement

### Deleting a Requirement

### Generating Traceability Diagram

## Frontend

## Testing

## Running the Application

## Future Improvements
