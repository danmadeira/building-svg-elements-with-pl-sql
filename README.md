## Building SVG Elements with PL/SQL

**Current Version:** `1.0.2` (26/10/2026)  
**License:** [GNU General Public License v3.0 (GPL-3.0)](https://opensource.org/licenses/GPL-3.0)

### Overview

`scalable_vector_graphics` is an Oracle **PL/SQL** package designed to programmatically build **Scalable Vector Graphics (SVG) version 1.1** documents and fragments based on the official W3C XML grammar.

PL/SQL (Procedural Language for SQL) is Oracle Corporation's procedural extension for SQL and the Oracle relational database. By encapsulating SVG 1.1 syntax into modular PL/SQL functions, this package enables developers to generate charts, diagrams, icons, and data-driven vector graphics directly inside the database—without relying on external rendering libraries.

Each function in the package returns a well-formed XML/SVG element—such as a document declaration, structural container, shape, text element, font definition, or gradient—complete with its respective attributes and presentation properties. Functions can be freely nested and composed both in procedural PL/SQL blocks and within standard SQL queries.

> **Note:** This project is under active development and does not yet cover the entire SVG 1.1 specification (such as filter effects or SMIL animations). However, it already implements the core structural containers, basic shapes, paths, text/font elements, gradients, and presentation attributes needed for most vector graphics use cases.

---

### Supported Elements & Functions

- **Document & Declarations:**
  - `xml10_declaration` (`<?xml ...?>`)
  - `document_type_declaration` (`<!DOCTYPE svg ...>` / `<!DOCTYPE html>`)
  - `svg_element` (`<svg>...</svg>`)
- **Descriptive & Metadata Elements:**
  - `title_element` (`<title>...</title>`)
  - `desc_element` (`<desc>...</desc>`)
- **Structural & Container Elements:**
  - `defs_element` (`<defs>...</defs>`)
  - `group_element` (`<g>...</g>`)
  - `symbol_element` (`<symbol>...</symbol>`)
  - `use_element` (`<use />`)
  - `a_element` (`<a>...</a>`)
  - `script_element` (`<script>...</script>`)
- **Basic Shapes, Paths & Images:**
  - `path_element` (`<path />`)
  - `rect_element` (`<rect />`)
  - `circle_element` (`<circle />`)
  - `ellipse_element` (`<ellipse />`)
  - `line_element` (`<line />`)
  - `polyline_element` (`<polyline />`)
  - `polygon_element` (`<polygon />`)
  - `image_element` (`<image>...</image>`)
- **Text & Font Elements:**
  - `text_element` (`<text>...</text>`)
  - `tspan_element` (`<tspan>...</tspan>`)
  - `tref_element` (`<tref />`)
  - `textpath_element` (`<textPath>...</textPath>`)
  - `font_element` (`<font>...</font>`)
  - `font_face_element` (`<font-face>...</font-face>`)
  - `font_face_src_element` (`<font-face-src>...</font-face-src>`)
  - `font_face_uri_element` (`<font-face-uri />`)
  - `font_face_name_element` (`<font-face-name />`)
  - `font_face_attribute` (font-face descriptor attributes)
  - `glyph_element` (`<glyph>...</glyph>`)
- **Gradients & Styling:**
  - `presentation_attribute` (`fill`, `stroke`, `opacity`, `font-*`, `text-*`, etc.)
  - `lineargradient_element` (`<linearGradient>...</linearGradient>`)
  - `radialgradient_element` (`<radialGradient>...</radialGradient>`)
  - `stop_element` (`<stop />`)

---

### Installation

Compile the package specification and body in your Oracle schema by running:

```sql
@src/ScalableVectorGraphics.sql
```

---

### PL/SQL Usage Examples

#### Example 1: Generating a Complete SVG Document in a PL/SQL Block

This example builds a complete standalone SVG 1.1 document containing a background rectangle, a styled circle, and a centered text label:

```sql
SET SERVEROUTPUT ON;

DECLARE
  l_svg_doc  CLOB;
  l_body     CLOB;
BEGIN
  -- Build the inner graphical elements
  l_body :=
    scalable_vector_graphics.title_element('Sample SVG Graphic') ||
    scalable_vector_graphics.desc_element('Generated with Oracle PL/SQL') ||
    scalable_vector_graphics.rect_element(
      p_x            => '10',
      p_y            => '10',
      p_width        => '380',
      p_height       => '180',
      p_rx           => '12',
      p_ry           => '12',
      p_presentation => scalable_vector_graphics.presentation_attribute(
                          p_fill         => '#f4f6f9',
                          p_stroke       => '#2c3e50',
                          p_stroke_width => '2'
                        )
    ) ||
    scalable_vector_graphics.circle_element(
      p_cx           => '200',
      p_cy           => '100',
      p_r            => '60',
      p_presentation => scalable_vector_graphics.presentation_attribute(
                          p_fill         => 'coral',
                          p_fill_opacity => '0.85',
                          p_stroke       => 'navy',
                          p_stroke_width => '3'
                        )
    ) ||
    scalable_vector_graphics.text_element(
      p_content      => 'PL/SQL + SVG 1.1',
      p_x            => '200',
      p_y            => '106',
      p_presentation => scalable_vector_graphics.presentation_attribute(
                          p_font_family => 'Arial, sans-serif',
                          p_font_size   => '18px',
                          p_font_weight => 'bold',
                          p_fill        => 'white',
                          p_text_anchor => 'middle'
                        )
    );

  -- Wrap the content with XML/DOCTYPE declarations and the root <svg> element
  l_svg_doc :=
    scalable_vector_graphics.xml10_declaration(
      p_standalone => 'no',
      p_encoding   => 'UTF-8'
    ) ||
    scalable_vector_graphics.document_type_declaration(p_doctype => 'S') ||
    scalable_vector_graphics.svg_element(
      p_content => l_body,
      p_width   => '400px',
      p_height  => '200px',
      p_viewbox => '0 0 400 200'
    );

  DBMS_OUTPUT.PUT_LINE(l_svg_doc);
END;
/
```

#### Example 2: Using Gradients (`<defs>`), Groups (`<g>`), and Polygons in PL/SQL

This example demonstrates how to define a reusable `<linearGradient>` inside a `<defs>` container and apply it to grouped shapes:

```sql
SET SERVEROUTPUT ON;

DECLARE
  l_defs     VARCHAR2(4000);
  l_group    VARCHAR2(4000);
  l_svg      CLOB;
BEGIN
  -- 1. Define a linear gradient with three color stops
  l_defs := scalable_vector_graphics.defs_element(
    p_elements => scalable_vector_graphics.lineargradient_element(
      p_id    => 'sunsetGrad',
      p_x1    => '0%',
      p_y1    => '0%',
      p_x2    => '100%',
      p_y2    => '100%',
      p_stops => scalable_vector_graphics.stop_element('0%',   '#ff512f', '1') ||
                 scalable_vector_graphics.stop_element('50%',  '#dd2476', '1') ||
                 scalable_vector_graphics.stop_element('100%', '#ff512f', '1')
    )
  );

  -- 2. Group a polygon and a path referencing the gradient
  l_group := scalable_vector_graphics.group_element(
    p_id           => 'star-group',
    p_transform    => 'translate(50, 20)',
    p_presentation => scalable_vector_graphics.presentation_attribute(
                        p_fill         => 'url(#sunsetGrad)',
                        p_stroke       => '#333333',
                        p_stroke_width => '2'
                      ),
    p_elements     => scalable_vector_graphics.polygon_element(
                        p_points => '100,10 40,198 190,78 10,78 160,198'
                      )
  );

  -- 3. Assemble the root <svg> element
  l_svg := scalable_vector_graphics.svg_element(
    p_content => l_defs || l_group,
    p_width   => '300px',
    p_height  => '240px',
    p_viewbox => '0 0 300 240'
  );

  DBMS_OUTPUT.PUT_LINE(l_svg);
END;
/
```

#### Example 3: Calling the Package Directly in SQL

Because all package functions return standard SQL types (`VARCHAR2` / `CLOB`) when using default boolean parameters, you can also invoke them directly inside a `SELECT` statement:

```sql
SELECT scalable_vector_graphics.svg_element(
         p_width   => '200px',
         p_height  => '200px',
         p_content => scalable_vector_graphics.circle_element(
                        p_cx           => '100',
                        p_cy           => '100',
                        p_r            => '80',
                        p_presentation => scalable_vector_graphics.presentation_attribute(
                                            p_fill         => 'coral',
                                            p_stroke       => 'navy',
                                            p_stroke_width => '4'
                                          )
                      )
       ) AS svg_doc
  FROM dual;
```

---

### References:

- BALES, D. J. *Beginning Oracle PL/SQL, 2nd Edition*. Apress, 2015.

- BELLAMY-ROYDS, A.; CAGLE, K. *SVG Colors, Patterns & Gradients: Painting Vector Graphics*. O'Reilly, 2016.

- BELLAMY-ROYDS, A.; CAGLE, K.; STOREY, D. *Using SVG with CSS3 and HTML5: Vector Graphics for Web Design, Second Release*. O'Reilly, 2018.

- EISENBERG, J. D.; BELLAMY-ROYDS, A. *SVG Essentials, Second Edition*. O'Reilly, 2015.

- JUNEAU, J.; ARENA, M. *Oracle and PL/SQL Recipes: A Problem-Solution Approach*. Apress, 2010.

- LIBBY, A. *Beginning SVG: A Practical Introduction to SVG using Real-World Examples*. Apress, 2018.

- MCDONALD, C. et col. *Mastering Oracle PL/SQL: Practical Solutions*. APress Media, LLC, 2004.

- MORIN, L. *Oracle Database PL/SQL Language Reference, 19c*. E96448-03, Oracle, August 2020.

- MURACH, J. *Murach's Oracle SQL and PL/SQL for Developers, 2nd Edition*. Mike Murach & Associates, 2014.

- ROSENZWEIG, B.; RAKHIMOV, E. S. *Oracle PL/SQL by example, 4th Edition*. Pearson Education, Inc., 2009.

- W3C *Scalable Vector Graphics (SVG) 1.1 (Second Edition)*, W3C Recommendation 16 August 2011. Available in: <https://www.w3.org/TR/SVG11/>
