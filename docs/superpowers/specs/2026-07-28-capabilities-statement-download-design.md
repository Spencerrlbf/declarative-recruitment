# Capabilities Statement Download Design

## Goal

Make `Capabilities Statement Update-design.pdf` the capabilities statement downloaded from the website while preserving the existing public filename and both current download links.

## Existing Behavior

- `index.html` contains two capabilities-statement download links.
- Both links target `Declarative-Recruitment-Capabilities-Statement.pdf`.
- The supplied replacement is a one-page PDF named `Capabilities Statement Update-design.pdf`.
- The replacement uses a clearer white-background layout and contains the intended company, procurement, service, experience, and contact information.

## Approved Design

Replace the bytes of `Declarative-Recruitment-Capabilities-Statement.pdf` with the bytes of `Capabilities Statement Update-design.pdf`.

Keep `index.html` unchanged. This preserves the existing public URL, avoids broken links or bookmarks, and makes both download buttons serve the replacement document immediately after deployment.

Keep the supplied source PDF in place and do not modify unrelated working-tree files.

## Data Flow

1. A visitor selects either "Download Capability Statement" link.
2. The existing HTML anchor requests `Declarative-Recruitment-Capabilities-Statement.pdf`.
3. The static host returns the newly replaced PDF.
4. The browser downloads the updated one-page capabilities statement.

## Failure Handling

- Stop if the supplied PDF is missing or is not recognized as a PDF.
- Stop if the public target filename cannot be replaced exactly.
- Stop if the source and target SHA-256 hashes differ after replacement.
- Stop if the final PDF cannot be rendered or has visual defects.
- Do not alter the download links unless verification shows that they no longer target the stable public filename.

## Verification

- Confirm the source and public target have identical SHA-256 hashes.
- Confirm the public target is recognized as a one-page PDF.
- Render the public target and visually inspect it for clipping, overlap, missing text, or unreadable content.
- Confirm both download anchors still target `Declarative-Recruitment-Capabilities-Statement.pdf`.
- Confirm the implementation changes only the intended public PDF asset, apart from the separately committed design and implementation documentation.

