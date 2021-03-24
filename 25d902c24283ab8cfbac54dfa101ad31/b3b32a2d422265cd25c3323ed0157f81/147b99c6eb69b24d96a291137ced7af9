import * as React from "react"
import { v4 as uuidv4 } from "uuid"

import Layout from "../components/layout"
import SEO from "../components/seo"

import $ from "jquery"

class SecondPage extends React.Component {

  async submit() {
    const body = JSON.stringify({
      id: uuidv4().toString(),
      header: $("#post-header").val(),
      body: $("#post-body").val(),
      timestamp: (new Date()).getTime()
    });
    await fetch(`https://fqpvlyf9q9.execute-api.us-east-2.amazonaws.com/prod/posts`, {
      body: body,
      method: "POST"
    })
    .then(res => res.json())
    .then(data => { console.log(data); });

    window.location.href = window.location.origin;
  }

  render() {
    return (
      <Layout>
        <SEO title="Write a Post" />
        <h1>Write a Post</h1>
        <p>Add your thoughts to the Journal.</p>
        <hr/>
        <input
          id="post-header"
          placeholder="Header..."
        />
        <br/><br/>
        <textarea
          id="post-body"
          style={{
            width: "100%",
            fontSize: "18pt"
          }}
          placeholder="Speak your mind..."
        >
        </textarea>
        <br/><br/>
        <button
          className="join-btn"
          onClick={this.submit}
        >Submit</button>
      </Layout>
    )
  }

}

export default SecondPage;
