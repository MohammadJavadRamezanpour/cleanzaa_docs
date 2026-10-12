# Cleanzza Platform Features and Workflows

This document describes the platform's main pages, registration and service workflows, payment process, and rules for customers and cleaners.

## Home Page Features

### 1 Cleaner Login and Registration

Cleaners can select this section to register or sign in to their accounts using their username and password.

### 2 Request Cleaning

This section is for users and customers—in other words, owners of homes, cars, pets, offices, swimming pools, or anything else requiring cleaning. Selecting this option takes them directly to the customer area. First-time customers verify their mobile number to create an account. The Phase 1 data policy is recorded in [ADR 0002](adr/0002-phase-1-personal-data.md).

The rest of the page should contain brand messaging, an explanation of how the platform works, descriptions of our services, and key statements that reassure users and encourage them to choose us.

## Cleaner Registration Page

On this page, the cleaner must:

1. Enter their first and last name, date and place of birth, current place of residence, full address, email address, and phone number.
2. Choose a username and password, then request verification.
3. Enter the verification code sent to their phone number.
4. Open the verification email and select the account-confirmation option.
5. Capture both sides of an ID card, or the required identity pages of a
   passport, live in the application. Provide work-authorization evidence where
   applicable.
6. Take and submit a live selfie using the application's camera.
7. Enter their fiscal or tax code.
8. Enter their bank account details.
9. Read and download the terms, select **I accept the terms**, and submit the application for review.

The cleaner also states whether they are employed, a student, or unemployed.
An employed person or student uploads supporting documentation.

The cleaner must then select the service areas in which they can work. A professional certificate must be uploaded for every service category except home cleaning. For home cleaning, the cleaner may identify themselves as either a beginner or a professional.

Next, the cleaner must indicate whether they own cleaning supplies and equipment, such as a mop or vacuum cleaner. If they do, they must list the equipment they have. Finally, they answer five psychological-assessment questions and receive a welcome message.

An authorized administrator manually checks identity and qualifications,
records the decision and reason, and approves the cleaner before they can
receive or accept jobs.

## Cleaner Login Page

The cleaner signs in using the username and password selected during registration.

## Customer Registration Page

Customers verify their mobile number with a short-lived code to create an
account. Before their first booking, they provide their name and service
address and accept the current terms; until then the account is pending
activation and cannot place an order. Payment information is collected by the
selected payment provider when needed. Birth details, tax code, identity-card
images, a selfie, and bank details are not routine customer registration
requirements; a separately approved need is required before collecting them.

## Customer Login Page

The customer enters their mobile number and receives a verification code. Entering that code allows them to sign in easily.

## Cleaner Profile

The cleaner first sees a control for indicating whether they are available for work. When they select **I am available for work**, they must take a selfie before entering the waiting queue.

The same page includes a score section and a list of actions the cleaner can take to improve their score. The score significantly affects access to future service requests: cleaners with lower scores see new job postings later than cleaners with higher scores.

The profile also includes:

- A service history showing all services the cleaner has completed.
- A payment history showing total earnings and commissions paid.
- Profile viewing and editing.
- Always-available online support chat.
- Approved completed-work highlights, ratings, and achievements. Home photos
  appear only after completion, customer satisfaction, administrator approval,
  and separate homeowner consent. Without photo consent, only non-private
  ratings and achievements appear. Consent can be withdrawn.

## Customer Profile

The page includes an **I need cleaning** option. After selecting it, the customer follows this process:

1. Specify whether the service is for themselves or someone else.
2. If it is for someone else, enter that person's first and last name and phone number.
3. Select one or more required cleaning services.
4. If home cleaning is selected, choose a beginner or professional cleaner.
5. Select the requested service date and time.
6. Specify which parts of the home need cleaning and how many rooms it has.
7. Where applicable, provide details such as the breed of their dog or cat or the model of their car.
8. Indicate whether cleaning equipment or supplies are required.
9. Start the search so the system can intelligently match the request with a cleaner.

The customer profile also includes:

- Payment and service histories.
- Profile editing.
- Live, 24-hour support chat.

## Orders Shown to Cleaners

The cleaner sees a continuously updated list of available services. Each listing
is removed as soon as another cleaner accepts it. Every listing shows the
service type, date, time, and options to accept or reject the request. The
customer name, exact address, phone number, and private images are hidden until
acceptance; the exact address is then shown only to the assigned cleaner.

If a cleaner accepts a service at a particular date and time, they cannot accept another order for the same time slot.

After a cleaner accepts an order, the customer has until the end of the cleaning period to pay. If payment is not made, the amount is automatically debited from the customer's account.

## Customer Rules

- Customers may book the same cleaner again, but they must pay a higher fee because repeat bookings reduce the platform's ability to create opportunities for new cleaners.
- To bring services into the platform and improve security, customers may introduce cleaners they already use outside the platform. The customer acts as the referrer and receives a commission for every service that cleaner completes through the platform.
- Customers may add funds to their account and pay from their account balance whenever needed.
- Customers have an emergency-call feature that allows them to contact the police immediately.
- Customers may submit a complaint within 24 hours after a cleaning service for review.
- Customers have a feature called **Peace of Mind**. Before the cleaner arrives, they use it to photograph dirty areas of their home. These photographs are required to submit a complaint; without before-service photographs, a complaint cannot be filed.
- If customers work with cleaners outside the platform, they accept the consequences, and the platform assumes no responsibility.
- Customers receive rewards when they refer other customers, based on the referred customers' orders and payments.

## After a Cleaner Accepts an Order

The customer sees the **Peace of Mind** feature. Activating it requires the customer to photograph and upload images of the dirty areas of the home. Doing so enables the right to submit a complaint about the cleaning. If the customer does not activate this feature, they cannot submit a complaint and accept responsibility for that decision.

## Cleaner Arrival at the Home or Service Location

As soon as the cleaner enters, the customer must confirm whether the cleaner's face matches their profile photograph. The customer then gives the cleaner the necessary instructions, and the work begins.

When the cleaner arrives, they must select **I have arrived** and take a selfie. This step prevents multiple orders from being handled through a single profile at the same time. After the selfie is taken, the system starts tracking the service time, and the cleaner performs the work.

## Completing the Cleaning Service

When the cleaning is complete, the cleaner selects **Complete**, uploads photographs of the cleaned areas, and specifies the settlement method. The cleaner hands the location back to the customer and leaves. Payment settlement may take up to two days, depending on the bank and the type of bank account.

Stored service photos are available only to authorized administrators and
case-scoped support. Raw photos are deleted seven days after completion unless
a documented complaint or legal hold applies. A separately approved,
anonymized publication copy may remain on the cleaner's profile while the
homeowner's photo consent remains active. See [ADR 0002](adr/0002-phase-1-personal-data.md).

## Payment

After the work is complete, the customer must pay either online or in cash.

- For an online payment, the platform deducts its commission and pays the remaining amount to the cleaner.
- For a cash payment, the customer has a limit of EUR 10 for settling the platform's commission. If the commission is not settled, the customer cannot continue using the system or select **Start work**. The system immediately finds another cleaner for the customer.

If settlement is not completed within 48 hours, the customer's account is blocked. To restore access, the customer must contact support. The customer also receives a negative score and is classified as having a poor payment record.

## Rating and Feedback

After an order is complete, the customer can rate the cleaner and submit feedback or criticism.

The customer can separately consent to publication of approved home photos
with the rating and satisfaction feedback. Without photo consent, the cleaner
may display only the rating and non-private achievements. The homeowner's
name, address, and contact details are never published.

## Cleaner Rules

Acceptance, rejection, dismissal, reinstatement, fines, and bonuses require a
recorded decision and reason from an authorized user. Allegations are reviewed
before sanctions or any referral to authorities; financial rules require a
separately approved policy.

- A cleaner may not take photographs outside the platform. A reported violation
  is reviewed before any score change or suspension.
- A cleaner must be punctual, clean, well-groomed, pleasant-smelling, and
  courteous. A reported violation is reviewed before any suspension or fine.
- A cleaner must comply with all customer-privacy requirements and other relevant rules so the customer's peace and safety are not put at risk.
- A cleaner who damages a customer's property must compensate the customer for the damage.
- A theft allegation triggers a restricted account and human investigation.
  A referral to authorities requires an applicable legal basis and review.
- A cleaner represents the company and must not engage in personal advertising.
  A reported violation is reviewed before any restriction or legal action.
- Service duration and quality may inform an approved fine or bonus only under
  a separately approved financial policy.

## Information changes and account exit

Users may edit ordinary profile details and request a copy or deletion of
their data. Correcting a verified identity record requires fresh SMS
verification, a new selfie, identification images, a signed request letter,
and manual review. A cleaner who leaves cannot accept new jobs. Their account
is restricted during a three-month review, with limited emails to former
clients about unresolved issues; ordinary account data is then deleted unless
an actual case or legal duty requires a documented hold. Customer deletion is
handled separately. The detailed policy and decisions to revisit are in
[ADR 0002](adr/0002-phase-1-personal-data.md).
