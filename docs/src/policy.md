# Data-Sharing Policy

We will assume that our users agree to our data-sharing policy that includes:

* For a user account, we will collect username (ID), the user's first and last names, and the user email and affiliation in our database.
The database records are kept confidential.
When uploading data, users can choose whether to disclose their username, affiliation, and email address in the data.

* To assure provenance and avoid uploading bad data, the repository requires access credentials to upload performance data and download data that requires access permission.

* Each performance data can have a different accessibility level: publicly available, private, or sharing with specific users/groups. Users can create specific user groups for data-sharing.

* GPTune will have an optional automatic synchronization mode, where the collected function evaluation data can be automatically uploaded to the repository.

* Users should not use automated scripts to collect very large performance data in the repository. We will limit the size of performance data that a user can upload/download at a time.

* Users have to manage their API keys securely, and have responsibility for any violations conducted using user API keys.

* There is a possibility that we can delete/alter/modify users' performance data for management purposes (generally we do not need to modify the performance results, but there is a possibility of changing the database structure).

# Plan to Maintain Security

This section discusses our efforts and plans to maintain security in the database.

**Secure HTTP.** To keep security, all requests, either using the crowd-tune API or the dashboard, to the repository must be done over HTTPs. The repository will not accept nonsecure HTTP requests.

**Database storage.** Our database, where all user and performance data is stored, is using  NERSC's storage and can only be accessed within a workload container at NERSC. In other words, any access to the storage is shielded by authenticating with a certain workload manager [Rancher2]("https://rancher.com/").

**User password and authentication.** A password is required to sign-in. We use Python Django's user authentication module, so we do not record the user's raw password in the database, but only a hash. Details about the Django authentication module can be found at [here](https://docs.djangoproject.com/en/3.2/topics/auth/default/).

**User API keys.** Since the API keys can be used instead of user passwords, we provide an option to use public and private key pairs for API keys. In this case, the user keeps the private key and we record only the public key in the database.

**Vulnerability scanning.** We regularly run a web vulnerability scanner (e.g. Arachni) to detect possible vulnerabilities.

