# Music_Store_Analysis

-- Senior most employee based on the job title

select top 1 title, first_name from Portfolio..employee
order by levels desc ;

-- which country has the most invoices

select * from Portfolio..invoice;


select count(*) as InvoiceCount , billing_country  
from Portfolio..invoice
group by  billing_country
order by 1 desc;

select top 3 total from Portfolio..invoice
order by 1 desc ;

-- same query using row_number
with cte as 
(select *, row_number() over(order by total desc ) as rn 
from Portfolio..invoice)
select * from cte where rn between 1 AND 3;

-- which is the best city?
select top 1 billing_city,sum(total) sumTotal from Portfolio..invoice
group by billing_city 
order by 2 desc;

-- who is the best customer
select * from Portfolio..invoice;

select top 1 c.first_name, max(i.total) maxSpending from Portfolio..customer c
INNER JOIN Portfolio..invoice i
ON c.customer_id = i.customer_id
Group by c.first_name
order by 2 desc;


-- Write query to return the email, first name, last name, & Genre of all Rock Music listeners. 
-- Return your list ordered alphabetically by email starting with A.

select * from Portfolio..customer;
select * from Portfolio..genre;

select DISTINCT c.email , c.first_name , c.last_name from Portfolio..customer c 
INNER JOIN Portfolio..invoice i on c.customer_id = i.customer_id
INNER JOIN Portfolio..invoice_line il on i.invoice_id = il.invoice_id
where track_id IN(
Select track_id from Portfolio..track t 
INNER JOIN Portfolio..genre g on t.genre_id = g.genre_id
where g.name like 'Rock')
order by email;

 --Let's invite the artists who have written the most rock music in our dataset. 
--Write a query that returns the Artist name and total track count of the top 10 rock bands.

select * from Portfolio..genre;
select * from Portfolio..track;



SELECT top 10 artist.artist_id, artist.name,COUNT(artist.artist_id) AS number_of_songs
FROM portfolio..track 
JOIN portfolio..album ON album.album_id = track.album_id
JOIN portfolio..artist ON artist.artist_id = album.artist_id
JOIN portfolio..genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id,artist.name
ORDER BY number_of_songs DESC;

-- Return all the track names that have a song length longer than the average song length. 
--Return the Name and Milliseconds for each track. 
-- Order by the song length with the longest songs listed first. */


select name , milliseconds from Portfolio..track
where milliseconds > 
(select AVG(milliseconds) from Portfolio..track)
;

--Find how much amount spent by each customer on artists?
--Write a query to return customer name, artist name and total sales 

select * from portfolio..customer;
select * from portfolio..artist;
select * from portfolio..invoice;
select * from portfolio..invoice_line;

with best_selling_artist as 
(select top 1 a.artist_id, a.name, sum(il.unit_price * il.quantity) totalsales 
from portfolio..invoice_line il
inner join portfolio..track t on t.track_id = il.track_id
inner join portfolio..album al on al.album_id = t.album_id
inner join portfolio..artist a on a.artist_id = al.artist_id
group by a.artist_id, a.name
order by 3 desc 
)

SELECT top 1 c.customer_id, c.first_name, c.last_name,
SUM(il.unit_price*il.quantity)
AS amount_spent
FROM portfolio..invoice i
JOIN portfolio..customer c ON c.customer_id = i.customer_id
JOIN portfolio..invoice_line il ON il.invoice_id = i.invoice_id
JOIN portfolio..track t ON t.track_id = il.track_id
JOIN portfolio..album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY c.customer_id, c.first_name, c.last_name,bsa.name
order by 4 desc;

-- Write a query that returns each country along with the top Genre

select * from Portfolio..customer;

WITH popular_genre AS 
(
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM portfolio..invoice_line 
	JOIN portfolio..invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN portfolio..customer ON customer.customer_id = invoice.customer_id
	JOIN portfolio..track ON track.track_id = invoice_line.track_id
	JOIN portfolio..genre ON genre.genre_id = track.genre_id
	GROUP BY customer.country, genre.name, genre.genre_id
	
	
)
SELECT * FROM popular_genre WHERE RowNo <= 1
