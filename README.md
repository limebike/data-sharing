# data-sharing

## Table of Contents

- [About](#about)
- [Authentication](#authentication)
- [MDS Extensions](#mds-extensions)
  - [Goals](#goals)
 
## About

[Top][toc]

## Authentication

[Top][toc]

## MDS Extensions

The MDS Provider API is the primary way that Lime shares data with governmental agencies 
and third parties. MDS Provider is a granular API, in that it shares individual trips and vehicle status changes, 
and doesn’t provide any kind of data aggregation or intelligence on top of such data.
Lime has developed API extensions to MDS Provider to share aggregated insights on top of MDS, 
which we call [MDS Extension](mds_extensions/README.md).

### Goals

- **Provide insights:** The data published through MDS Extensions provides insights on top of Lime’s data. This will 
help improve MDS from its current form.

    Primarily, providing data insights will allow Lime to report more accurate operational information to agencies. 
    Certain calculations such as the number of vehicles currently available in the fleet at a given time need to be 
    calculated by the agency via the status changes feed. Issues with this process frequently arise: each agency 
    performs the calculation differently, the process is opaque to Lime, and agencies make unfounded assumptions 
    about the data in the feed.

    As the operator, Lime can provide this data more accurately, following the same process for every city, while 
    understanding the nuances of the dataset we maintain. This will create a predictable, accurate process 
    that will result in uniformity across cities and fewer questions and complaints about common processes.

- **Data protection:** Users consider granular trips data and individual vehicle status change feed to reflect personal
or otherwise sensitive information.  In particular, this data, when combined with other datasets, can allow a city to
identify individual users, their movement patterns, and details such as a user's place of worship.  Those with access
to this information, including cities and agencies, could be subject to the **GDPR**, the **CCPA**, and 
other data privacy and security regulations and standards.
    
    MDS Extensions helps mitigate these concerns while continuing to allow agencies the ability to monitor permit
    compliance and understand traffic patterns, among other understood agency objectives.  To do so, MDS Extension, as
    opposed to MDS Provider, aggregates granular data by applying [k-anonymity](https://en.wikipedia.org/wiki/K-anonymity)
    principles.  The aggregated feed allows agencies to receive data without needing to worry about data privacy or
    security

[Top][toc]
