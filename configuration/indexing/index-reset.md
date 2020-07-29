# Index reset

Index reset feature deletes the chemical index from the ChemLocator database.

It does *NOT*:

  - modify or delete the registered content sources
  - modify or delete the application settings
  - reset or upgrade the ChemLocator database
  - modify or delete the indexed documents in any way (no data loss!)



!!!note
    It is only possible to reset the index when all the content sources are in idle 
    state.


!!!warning "Not reversible"
    Once the index reset was completed, the process can not be reverted, and the 
    index data is permanently removed from the database. The content sources will 
    need to be re-indexed to recreate the data. Again, the source documents remain 
    untouched.
