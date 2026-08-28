
export default async function handler(req, res) {
  if (req.method !== "POST") {
    return res.status(405).json({
      error: "Method tidak diperbolehkan."
    });
  }

  try {
    const token = process.env.VERCEL_TOKEN;

    if (!token) {
      return res.status(500).json({
        error: "VERCEL_TOKEN belum dipasang."
      });
    }

    const { projectName, html } = req.body || {};

    if (!projectName || !html) {
      return res.status(400).json({
        error: "Nama project dan HTML wajib diisi."
      });
    }

    const name = String(projectName)
      .toLowerCase()
      .trim()
      .replace(/[^a-z0-9-]/g, "-")
      .replace(/-+/g, "-")
      .replace(/^-+|-+$/g, "")
      .substring(0, 50);

    if (!name) {
      return res.status(400).json({
        error: "Nama project tidak valid."
      });
    }

    if (Buffer.byteLength(html, "utf8") > 2 * 1024 * 1024) {
      return res.status(400).json({
        error: "HTML terlalu besar. Maksimal 2 MB."
      });
    }

    const response = await fetch(
      "https://api.vercel.com/v13/deployments",
      {
        method: "POST",
        headers: {
          "Authorization": `Bearer ${token}`,
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          name,
          files: [
            {
              file: "index.html",
              data: html
            }
          ],
          projectSettings: {
            framework: null
          }
        })
      }
    );

    const data = await response.json();

    if (!response.ok) {
      return res.status(response.status).json({
        error:
          data?.error?.message ||
          "Vercel API menolak deployment."
      });
    }

    return res.status(200).json({
      success: true,
      url: `https://${data.url}`,
      deploymentId: data.id
    });

  } catch (error) {
    console.error(error);

    return res.status(500).json({
      error: "Server mengalami kesalahan."
    });
  }
}
