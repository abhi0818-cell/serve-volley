# Smash & Volley: court booking (Supabase version)

`index.html` is the whole site: one file, no build step. It talks to the Supabase project **pickleball-booking** (`hzgsdpemrzrxwtuiodbq`).

## What the database guarantees
- A court cannot be booked twice for overlapping times. Postgres refuses the second booking itself, so this holds for the website, the front desk and any future outside source.
- Players cannot write bookings directly. They call `create_booking`, which checks opening hours, sport, length, the 14-day window and the 4-upcoming-bookings limit, and sets the price on the server.
- Staff use `admin_create_booking` and can block courts for maintenance. Blocks and bookings can never overlap either.
- Cancelling goes through `cancel_booking` (players until 2 hours before start, staff any time).
- Every create and cancel is recorded in `booking_events`.
- Users cannot make themselves admin.

## To go live
1. **Host it.** Upload this folder to any static host (Netlify, Vercel, Cloudflare Pages, or GitHub Pages).
2. **Allow your site address for sign-in.** Supabase dashboard > Authentication > URL Configuration: set Site URL to your live address and add it under Redirect URLs.
3. **Set up email sending.** Supabase's built-in email is limited to a few messages an hour. Before launch add your own SMTP provider (Authentication > Emails > SMTP settings), or sign-in links will fail for real players.
4. **Create your admin login (no email needed).** In Supabase open Authentication > Users > Add user > Create new user. Enter your email and a password, and tick "Auto Confirm User". Then, in the SQL editor, run:
   `update public.profiles set is_admin = true where id = (select id from auth.users where email = 'YOUR@EMAIL');`
   Sign in on the site with that email and password and the Master schedule tab appears. Other staff: they create an account on the site (Sign in > Create an account), then run the same command for their email.
5. **Check the numbers.** Sign in as admin and open the **Settings** tab. There you set the time zone, peak start time, longest booking, upcoming-bookings limit, cancellation cutoff, days bookable ahead, the four hourly rates (pickleball and badminton, off-peak and peak), and each court's name, hours and on/off. Changes apply to new bookings straight away; existing bookings keep their price.

## Signing in
Players can sign in with email and password, create an account, reset a forgotten password, or ask for an emailed sign-in link. Reset and link emails go through Supabase's built-in email, which only delivers to your Supabase team's addresses. Connect your own email service (Authentication > Emails > SMTP settings) before real players use them. Google sign-in can be added later.

## Not built yet
- Card payments (bookings are confirmed and paid at the club).
- Confirmation emails.
- Playtomic and CourtReserve: they would send bookings in through a Supabase Edge Function that uses the `source` and `external_id` columns, so repeated deliveries are ignored.
