# Super Awesome Quizzes

### Installation
1. Clone this repository using your favourite method, such as `gh repo clone Xenuku/super-awesome-quiz`
2. Open the cloned folder in the terminal and run `npm install` to install dependencies
3. Use `npm run dev -- --open` to run the application and open it in your browser

### Design Choices
I decided to go for a 'streak' style application, where you constantly get new quizzes to see if you can build up a streak of correct answers. I was originally going to do a set of 10, and submit all the answers at once to return a score however I liked the idea of getting instant feedback for each answer submitted.

I chose to just focus on the one error mentioned about the API being insanely popular, and therefore did not do any automatic retrying of requests as to not overload the server. I left it up to the user to process their request again and caught just the one error.

I wanted this to be a simple and colourful application that works on both mobile and desktop.