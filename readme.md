

```ts
{
  collectionName?: string,
  isActive: boolean,
  isIndexed?: boolean,
  mimeType?: string,
  name?: string,
  uri: string,
}
```
### Create the DocumentManager Component

```js
import React, { useState } from 'react';

const DocumentManager = ({ documents, onDocumentsChange }) => {
  const [newDocumentName, setNewDocumentName] = useState('');
  const [newDocumentURI, setNewDocumentURI] = useState('');
  const [newDocumentIsActive, setNewDocumentIsActive] = useState(true);

  const handleAddDocument = () => {
    const newDocument = {
      name: newDocumentName,
      uri: newDocumentURI,
      isActive: newDocumentIsActive,
      mimeType: 'application/pdf', // Example, you might want to determine this based on the file or URL
    };
    onDocumentsChange([...documents, newDocument]);
    setNewDocumentName('');
    setNewDocumentURI('');
    setNewDocumentIsActive(true); // Reset to default active state
  };

  const handleRemoveDocument = (index) => {
    // Instead of removing, we deactivate
    const updatedDocuments = documents.map((doc, i) => i === index ? { ...doc, isActive: false } : doc);
    onDocumentsChange(updatedDocuments);
  };

  const toggleDocumentActiveState = (index) => {
    const updatedDocuments = documents.map((doc, i) => {
      if (i === index) {
        return { ...doc, isActive: !doc.isActive };
      }
      return doc;
    });
    onDocumentsChange(updatedDocuments);
  };

  return (
    <div>
      <input
        type="text"
        placeholder="Document Name"
        value={newDocumentName}
        onChange={(e) => setNewDocumentName(e.target.value)}
      />
      <input
        type="text"
        placeholder="Document URI"
        value={newDocumentURI}
        onChange={(e) => setNewDocumentURI(e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={newDocumentIsActive}
          onChange={(e) => setNewDocumentIsActive(e.target.checked)}
        />
        Active
      </label>
      <button onClick={handleAddDocument}>Add Document</button>
      <ul>
        {documents.map((doc, index) => (
          <li key={index}>
            {doc.name} ({doc.mimeType}) - <a href={doc.uri} target="_blank" rel="noopener noreferrer">Open</a>
            <label>
              <input
                type="checkbox"
                checked={doc.isActive}
                onChange={() => toggleDocumentActiveState(index)}
              />
              Active
            </label>
            <button onClick={() => handleRemoveDocument(index)}>Remove</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default DocumentManager;
```

### Update the ScenarioSettings Component


```js
import React, { useState, useEffect } from 'react';
import DocumentManager from '../../components/DocumentManager';

const ScenarioSettings = ({
  scenarioDocuments,
  updateScenarioDocuments,
  // other props
}) => {
  // Use state to manage the local copy of scenario documents
  const [documents, setDocuments] = useState([]);

  // Update local state when scenarioDocuments prop changes
  useEffect(() => {
    setDocuments(scenarioDocuments || []);
  }, [scenarioDocuments]);

  // Handler to update documents both locally and in the parent component
  const handleDocumentsChange = (updatedDocuments) => {
    setDocuments(updatedDocuments);
    updateScenarioDocuments(updatedDocuments);
  };

  return (
    <div>
      {/* Other settings UI */}
      <DocumentManager
        documents={documents}
        onDocumentsChange={handleDocumentsChange}
      />
    </div>
  );
};

export default ScenarioSettings;
```

### Pass Props in the Parent Compone

```js
import React, { useState } from 'react';
import ScenarioSettings from './Settings';

const ScenarioEditor = () => {
  // State to manage the entire scenario, including documents
  const [scenario, setScenario] = useState({
    // initial scenario state, including documents
    documents: [],
  });

  // Function to update scenario documents
  const updateScenarioDocuments = (updatedDocuments) => {
    setScenario((prevScenario) => ({
      ...prevScenario,
      documents: updatedDocuments,
    }));
    // Here, you would also handle updating the scenario in your backend or database
  };

  return (
    <div>
      <ScenarioSettings
        scenarioDocuments={scenario.documents}
        updateScenarioDocuments={updateScenarioDocuments}
        // pass other necessary props
      />
    </div>
  );
};

export default ScenarioEditor;
```
