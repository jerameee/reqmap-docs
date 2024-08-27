# Overview

ReqMap is a Flask-based web application for managing and visualizing requirements. It allows users to create, update, and delete requirements, as well as view a traceability diagram of the requirements.

## Architecture

The application follows a Model-View-Controller (MVC) architecture:

* __Model:__ Defined in ```models.py```
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

The core data model is the ```Requirement``` class, defined in ```models.py```

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

Endpoint: `PUT/requirements/<init:id>`

``` py
@app.route('/requirements/<int:id>', methods=['PUT'])
def update_requirement(id):
    data = request.get_json()
    req = Requirement.query.get_or_404(id)
    req.title = data['title']
    req.description = data['description']
    req.priority = data['priority']
    req.status = data['status']
    req.version += 1
    db.session.commit()
    return jsonify({'message': 'Requirement updated successfully!'})
```

### Deleting a Requirement

Endpoint: `PUT/requirements/<init:id>`

``` py
@app.route('/requirements/<int:id>', methods=['DELETE'])
def delete_requirement(id):
    req = Requirement.query.get_or_404(id)
    db.session.delete(req)
    db.session.commit()
    return jsonify({'message': 'Requirement deleted successfully!'})
```

### Generating Traceability Diagram

Endpoint: `GET /requirements/trace`

```py
@app.route('/requirements/trace', methods=['GET'])
def trace_requirements():
    requirements = Requirement.query.all()
    graph = nx.DiGraph()

    for req in requirements:
        graph.add_node(req.id, label=req.title)
        if req.parent_id:
            graph.add_edge(req.parent_id, req.id)

    plt.figure(figsize=(12, 8))
    pos = nx.spring_layout(graph)
    nx.draw(graph, pos, with_labels=True, arrows=True, node_size=3000, node_color='lightblue')

    labels = nx.get_node_attributes(graph, 'label')
    nx.draw_networkx_labels(graph, pos, labels, font_size=8)

    output = BytesIO()
    plt.savefig(output, format='png')
    plt.close()
    output.seek(0)
    return send_file(output, mimetype='image/png')
```

This function generates a directed graph visualization of the requirements, showing their hierarchical relationships.

## Frontend

The frontend is a simple HTML page (`index.html`) styled with CSS (`styles.css`). It provides a form for adding new requirements and displays existing requirements.

## Testing

Unit tests are provided in `test_app.py`. Here's an example test case:

```py
def test_create_requirement(self):
    response = self.app.post('/requirements', json={
        'title': 'Test Requirement',
        'description': 'This is a test.',
        'priority': 'High',
        'status': 'Open'
    })
    self.assertEqual(response.status_code, 201)
    self.assertIn('Requirement created successfully!', response.get_data(as_text=True))
```

## Installation

1. Clone the repository:

   ``` md
   git clone https://github.com/your-username/reqmap.git

   cd reqmap

   ```

2. Create a virtual environment and activate it:

   ``` md
   python -m venv venv
   source venv/bin/activate  # On Windows, use `venv\Scripts\activate`

   ```

3. Install the required packages:

   ``` md
   pip install -r requirements.txt

   ```

## Running the Application

1. Start the Flask development server:
   ```
   python app.py
   ```

2. Open a web browser and navigate to `http://localhost:5000`

## Usage

- Add new requirements using the form on the main page
- View existing requirements in the list below the form
- Update or delete requirements by implementing the necessary frontend functionality (not included in the current version)
- Click on "View Traceability Diagram" to see the relationships between requirements

## Running the Application

To run the application:

1. Install dependencies: `pip install -r requirements.txt`
2. Run the Flask application: `python3 app.py`

The application will be available at `http://localhost:5000`

## Future Improvements

* Implement user authentication and authorization
* Add pagination for large numbers of requirements
* Implement more advanced filtering and sorting options
* Enhance the traceability diagram with more interactive features