import { useState, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Inbox } from "lucide-react";

export default function Eric({ onFileUploaded, fetchLegalForms }) {
  const [files, setFiles] = useState([]);
  const [uploading, setUploading] = useState(false);
  const [uploadStatus, setUploadStatus] = useState("");
  const [uploadedFiles, setUploadedFiles] = useState([]);
  const [state, setState] = useState("");
  const [formType, setFormType] = useState("");
  const [legalForms, setLegalForms] = useState([]);

  useEffect(() => {
    const storedFiles = localStorage.getItem("uploadedFiles");
    if (storedFiles) {
      setUploadedFiles(JSON.parse(storedFiles));
    }
  }, []);

  const handleFileChange = (event) => {
    const selectedFiles = Array.from(event.target.files);
    setFiles([...files, ...selectedFiles]);
  };

  const handleUpload = async () => {
    if (files.length === 0) {
      setUploadStatus("Please select files to upload.");
      return;
    }
    setUploading(true);
    setUploadStatus("Uploading...");
    const formData = new FormData();
    files.forEach((file) => formData.append("files", file));

    try {
      const response = await fetch("/api/upload", {
        method: "POST",
        body: formData,
      });
      const result = await response.json();
      console.log("Upload Success:", result);
      setUploadStatus("Upload Successful!");
      setFiles([]);
      const newUploadedFiles = [...uploadedFiles, ...files.map(file => file.name)];
      setUploadedFiles(newUploadedFiles);
      localStorage.setItem("uploadedFiles", JSON.stringify(newUploadedFiles));
      if (onFileUploaded) {
        onFileUploaded(result);
      }
    } catch (error) {
      console.error("Upload Error:", error);
      setUploadStatus("Upload Failed. Try again.");
    } finally {
      setUploading(false);
    }
  };

  const handleFetchLegalForms = async () => {
    if (!state || !formType) {
      setUploadStatus("Please enter a state and form type to fetch legal forms.");
      return;
    }
    try {
      const response = await fetch(`/api/legal-forms?state=${state}&type=${formType}`);
      const result = await response.json();
      setLegalForms(result.forms);
    } catch (error) {
      console.error("Error fetching legal forms:", error);
      setUploadStatus("Failed to retrieve legal forms.");
    }
  };

  return (
    <Card className="p-4 max-w-lg mx-auto mt-10 border-dashed border-2 border-gray-300 rounded-2xl">
      <CardContent className="flex flex-col items-center">
        <label
          className="flex flex-col items-center p-10 w-full cursor-pointer bg-gray-100 hover:bg-gray-200 rounded-lg"
        >
          <Inbox size={48} className="text-gray-500" />
          <span className="mt-2 text-gray-700">Drag & Drop Files Here</span>
          <input type="file" multiple className="hidden" onChange={handleFileChange} />
        </label>
        <ul className="mt-4 text-sm text-gray-600">
          {files.map((file, index) => (
            <li key={index}>{file.name}</li>
          ))}
        </ul>
        <Button onClick={handleUpload} className="mt-4" disabled={uploading}>
          {uploading ? "Uploading..." : "Upload Files"}
        </Button>
        {uploadStatus && <p className="mt-2 text-gray-700">{uploadStatus}</p>}
        <div className="mt-6 w-full">
          <h3 className="text-lg font-semibold text-gray-700">Previously Uploaded Files:</h3>
          <ul className="mt-2 text-sm text-gray-600">
            {uploadedFiles.map((file, index) => (
              <li key={index}>{file}</li>
            ))}
          </ul>
        </div>
        <div className="mt-6 w-full">
          <h3 className="text-lg font-semibold text-gray-700">Get Legal Forms</h3>
          <input
            type="text"
            placeholder="Enter State (e.g., California)"
            value={state}
            onChange={(e) => setState(e.target.value)}
            className="mt-2 p-2 border rounded w-full"
          />
          <input
            type="text"
            placeholder="Enter Form Type (e.g., Divorce, Tenant Rights)"
            value={formType}
            onChange={(e) => setFormType(e.target.value)}
            className="mt-2 p-2 border rounded w-full"
          />
          <Button onClick={handleFetchLegalForms} className="mt-4">
            Fetch Legal Forms
          </Button>
          <ul className="mt-2 text-sm text-gray-600">
            {legalForms.map((form, index) => (
              <li key={index}>{form}</li>
            ))}
          </ul>
        </div>
      </CardContent>
    </Card>
  );
}