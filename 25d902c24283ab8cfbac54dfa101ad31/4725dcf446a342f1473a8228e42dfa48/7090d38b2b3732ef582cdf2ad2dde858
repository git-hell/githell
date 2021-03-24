import * as React from "react"
import PropTypes from "prop-types"
import { Link } from "gatsby"
import { StaticImage } from "gatsby-plugin-image"

import "@fontsource/gabriela" // Defaults to weight 400 with all styles included.

const Header = ({ siteTitle }) => (
  <header
    style={{
      marginBottom: `1.45rem`,
    }}
  >
    <div
      style={{
        margin: `0 auto`,
        maxWidth: 960,
        padding: `1.45rem 1.0875rem`,
      }}
    >
      <h1 style={{ margin: 0 }}>
        <Link
          to="/"
          id="site-header"
          style={{
            textDecoration: `none`,
            fontFamily: `Gabriela, sans-serif`,
          }}
        >
          <StaticImage
            src="../images/AltoLogoWithBackground.png"
            width={43}
            height={43}
            style={{
              marginRight: 20,
              borderRadius: ".25rem",
              verticalAlign: "bottom",
            }}
          />
          {siteTitle}
        </Link>
        <Link
          to="https://alto.sh"
          style={{
            textDecoration: `none`,
            fontFamily: `Gabriela, sans-serif`,
          }}
        >
          <span style={{ color: "#666", fontSize: "1.4rem" }}> by Alto</span>
        </Link>
      </h1>
    </div>
  </header>
)

Header.propTypes = {
  siteTitle: PropTypes.string,
}

Header.defaultProps = {
  siteTitle: ``,
}

export default Header
