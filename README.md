## Connect Your React App with Flask (Python) Backend

Tell me if this sounds familiar. Your have been developing on frontend using React for a while and just learned one of the most popular languages and frameworks for backend, aka. Python with Flask. You just can't wait to whip them together for your new awesome app and take that full stack crown... but, how? Are you lost in a sea of online resources? Well, hope you wouldn't be anymore after reading this tutorial. Let's get everything set up, shall we?

### The Simplest Method in a Nutshell

Create RESTful API using Flask then have React make the API calls

There are layers in even the simplest full stack apps, with compiled bundle files on the top (assuming you are familiar with Webpack), React source files in the middle and backend using Python and Flask at the bottom. The frontend and backend are essentially separated from each other. Backend will run first, then frontend. We will start with the bottom layer.

<p align="left">
  <img src="/images/layer.png" width="400" />
</p>

### Set Up Flask

The example I use here is a simple movie rating app. You can add movie title and your own by marking stars. I won't go into detail on how the API is created step by step. For a complete tutorial on this, I recommend [this course](https://www.udemy.com/course/rest-api-flask-and-python/).

We're going to install Flask and SQLAlchemy first by running:

```
pip install flask flask-sqlalchemy
```

Under the `api` folder, we will make a `__init__.py` file and start with some configurations. SQLite is being used as the database here since it's a simple application. The file will be like this:

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
def create_app():
    app = Flask(__name__)
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
    db.init_app(app)
    from .views import main
    app.register_blueprint(main)
    return app
```

There will be two endpoints for this API: for viewing all the movies and adding a new movie. The `view.py` file will be like this:

```python
from flask import Blueprint, jsonify, request
from . import db
from .models import Movie

main = Blueprint('main', __name__)
@main.route('/add_movie', methods=['POST'])
def add_movie():
    movie_data = request.get_json()
    new_movie = Movie(title=movie_data['title'], rating=movie_data['rating'])
    db.session.add(new_movie)
    db.session.commit()
    return 'Done', 201

@main.route('/movies')
def movies():
    movie_list = Movie.query.all()
    movies = []
    for movie in movie_list:
        movies.append({'title' : movie.title, 'rating' : movie.rating})
    return jsonify({'movies' : movies})
```

To reflect what data we post and fetch from the `view.py`, let's create the database table in the `model.py` file:

```python
from . import db

class Movie(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(50))
    rating = db.Column(db.Integer)
```

- With these, the simplest API is efficiently done! But hold that high-five. Testing comes first. In the root folder, type this in the command line:

```
FLASK_APP= api
```

- This command points the server to our app folder. Then run:

```
flask run
```

You should see the URL with the port number as below:

<p align="left">
  <img src="/images/port.png" width="600" />
</p>

- Now we move on to create the app and database. Type `python` at the command line. When prompted, run:

```
from api.models import Movie
from api import db, create_app
db.create_all(app=create_app())
```

Go back to the `api` folder, you should see a `database.db` file was created.

<p align="left">
  <img src="/images/db_created.png" width="200" />
</p>

- To verify that this file has the `movie` table we want, type at command line:

```
sqlite3 api/database.db,
```

Then:

```
.tables
```

You should see `movie` shows up.

Now let's see some real data getting posted and retrieved. Open up Postman and use the URL from `flask run`, which most likely will be `http://127.0.0.1:5000`. Through the `add_movie` route, let's add a new movie in JSON format:

```javascript
{"title": "Avengers: End Game", "rating": 5}
```

How do we know it worked? You guessed it right. We check our `database.db` file again. After the same command as above, type:

```
select * from movie
```

You should see the movie we just created appear:

<p align="left">
  <img src="/images/sql.png" width="250" />
</p>

Add a few more movies of your choice, and make sure you see them when you send `GET` request through `movies` route.

That concludes the backend/API part of this tutorial. Once we connect to React, you will be able to see them on your app so proudly. What we are going to do next is adding the movie component to the frontend then send requests through React to our Flask app. Let's dive in!

### The React Layer

Now we move on to React, it's a whole new world. First, make sure you have `npm` installed along with `npx`. Let's start with creating a boilerplate project by running:

```
npx create-react-app <YOUR NAME OF THE PROJECT>
```

Don't know about `create-react-app`? Well, you should. It's one of the most efficient and painless ways to start a React project. To get it running, type in the command line:

```
npm start
```

You should see this:

<p align="left">
  <img src="/images/compile.png" width="400" />
</p>

Go to the localhost:3000 (might not be 3000 on your end):

<p align="left">
  <img src="/images/react.png" width="400" />
</p>

If we get to this far, you're ready to write something awesome in your React app. You can choose any UI for your app: Bootstrap, Material UI, Semantic UI, etc. Install and import them in your `index.js`. We're using Semantic UI here.

```javascript
import "semantic-ui-css/semantic.min.css";
```

Most of the work we are going to do is in the `App.js` file. We can clear out the placeholder code inside the `App` div and draft our requests, starting with rendering the movie list. We use the [React Effect Hook](https://reactjs.org/docs/hooks-effect.html):

```javascript
function App() {
  const [movies, setMovies] = useState([]);
  useEffect(() => {
    fetch("/movies").then(response =>
      response.json().then(data => {
        setMovies(data.movies);
      })
    );
  }, []);

  return <Container style={{ marginTop: 40 }}></Container>;
}
```

We need to add a `proxy` field to `package.json` to tell the server to proxy unknown requests to the Flask API in development:

```javascript
"proxy": "http://localhost:3000",
```

Do keep in mind that if you use old-fashion `Promise` inside the Effect Hook, it would throw an error. That's the reason we use `fetch` here.

If you're completely lost with the code above, I highly recommend you [this course](https://www.udemy.com/course/react-the-complete-guide-incl-redux/) for a refresher on React and Hooks.

Now we have the movie data from the API, we will need a component to list them out. So let's create a new folder called `components`, and inside, we will have a `Movie.js` file. It will look like this:

```javascript
export const Movies = ({ movies }) => {
  return (
    <List>
      {movies.map(movie => {
        return (
          <List.Item key={movie.title}>
            <Header>{movie.title}</Header>
            <Rating rating={movie.rating} maxRating={5} disabled />
          </List.Item>
        );
      })}
    </List>
  );
};
```

`List` here comes from Semantic UI. And don't forget to add a `key` to each item. This is a must in React.

Switch back to `App.js`, add the `Movie` component inside the App.

```javascript
return (
  <div className="App">
    <Movies />
  </div>
);
```

Inside the browser, you should see all the movies you posted via Postman getting rendered here.

<p align="left">
  <img src="/images/movies.png" width="750" />
</p>

The next step is to take care of the part where we add new movies. Make a new component file inside the `components` folder. We name it `MovieForm` in this example, which ends up like this:

```javascript
export const MovieForm = ({ onNewMovie }) => {
  const [title, setTitle] = useState("");
  const [rating, setRating] = useState(1);

  return (
    <Form>
      <Form.Field>
        <Input
          placeholder="movie title"
          value={title}
          onChange={e => setTitle(e.target.value)}
        />
      </Form.Field>
      <Form.Field>
        <Rating
          icon="star"
          rating={rating}
          maxRating={5}
          onRate={(_, data) => {
            setRating(data.rating);
          }}
        />
      </Form.Field>
      <Form.Field>
        <Button
          onClick={async () => {
            const movie = { title, rating };
            const response = await fetch("/add_movie", {
              method: "POST",
              headers: {
                "Content-Type": "application/json"
              },
              body: JSON.stringify(movie)
            });

            if (response.ok) {
              console.log("response worked!");
              onNewMovie(movie);
              setTitle("");
              setRating(1);
            }
          }}
        >
          submit
        </Button>
      </Form.Field>
    </Form>
  );
};
```

I know this looks like an intimating big chunk of code, but if you know React, you would know what I did here is merely:

- giving users the input field to type in a new movie and selecting a rating
- setting the `state` to the input value;
- adding a submit button, which sends the data to the `add_moive` route using `POST` method

The `onNewMovie` function is added in the `App.js` and passed to `MovieForm` as a prop, which keeps the `currentMovies` and gets updated with new movies.

```javascript
<MovieForm
  onNewMovie={movie => setMovies(currentMovies => [movie, ...currentMovies])}
/>
```

Then we set the form back to empty with `setTitle("")` and `setRating(1)`.

With this, we're done with the React layer as well! Don't forget to go to the browser, type some movies and submit them. See if they are rendered to the list properly.

That's a wrap! If everything goes without a hinge, you have, in your hands, a fully-connected Flask + React app. It isn't that hard at all, is it? A bigger scalable app will require much more resources and efforts of course, but this is a good place to start and expand your knowledge from.

Before you go, let's take one last review at our app from a higher level by looking at the file structure.

<p align="left">
  <img src="/images/folder.png" width="250" />
</p>

We have the Flask app live in the `server` folder, while React in the `static`. They each have their separate configurations and exist in a fairly independent way. The connection between them is built by calling APIs from React to Flask app and passing along data back and forth.

I hope you now have a clear idea of how to connect Flask with React app and structure your code. I'm constantly learning and I hope you are too, from this tutorial. Thank you for reading!
