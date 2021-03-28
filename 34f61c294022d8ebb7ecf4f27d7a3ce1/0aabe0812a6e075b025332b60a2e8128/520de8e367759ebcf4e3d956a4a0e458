import * as React from "react"
import { Link } from "gatsby"

import Layout from "../components/layout"
import SEO from "../components/seo"

class IndexPage extends React.Component {

  constructor(props) {
    super(props);

    this.state = {
      posts: []
    }
  }

  async componentDidMount() {
    const data = await fetch(`https://fqpvlyf9q9.execute-api.us-east-2.amazonaws.com/prod/posts`);
    const posts = await data.json();
    this.setState({ posts: posts });
  }

  render () {
    return (
      <Layout>
        <SEO title="Ethica" />
        <br/>
        <p
          style={{
            fontFamily: "Gabriela, sans-serif",
            fontSize: "22pt",
            fontWeight: "bold",
            color: "#666",
            lineHeight: "1.4",
          }}
        >
          Thoughts on Computing Ethics.
        </p>
        <br/>
        <Link 
          to="/contribute/"
        >
          <button
            className="join-btn"
            style={{
              fontFamily: "Gabriela, sans-serif"
            }}
          >
            Write a Post
          </button>
        </Link>
        &nbsp;&nbsp;
        <Link 
          to="https://airtable.com/shrYonv1BROnD6Yc9"
          target="_blank"
          rel="noreferrer"
        >
          <button
            className="join-btn"
            style={{
              fontFamily: "Gabriela, sans-serif"
            }}
          >
            Join Alto
          </button>
        </Link>
        <br/><br/>
        <hr/>
        <br/>
        <h1>Posts</h1>
        <br/>
        {
          this.state.posts.map(p => (
            <>
              <Link to="/" className="post-link">
                <h1>📃 {p.header.S}</h1>
              </Link>
              <h4>
                  {(new Date(parseInt(p.timestamp.N))).toDateString()}
              </h4>
              <p
                style={{
                  fontFamily: "Gabriela, sans-serif",
                  fontSize: "18pt",
                  lineHeight: 1.3
                }}
              >
                {p.body.S}
              </p>
              <br/>
            </>
          ))
        }
      </Layout>
    )
  }
}

export default IndexPage;
