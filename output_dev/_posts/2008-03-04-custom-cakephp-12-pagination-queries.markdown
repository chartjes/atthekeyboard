<hr />

<p>layout: post
title: Custom CakePHP 1.2 Pagination Queries
date: 2008-03-04</p>

<h2>author: Chris Hartjes</h2>

<p>
(Note: I will be cross-posting a version of this to <a href='http://book.cakephp.org'>the CakePHP Cookbook</a> within a day or two).
</p>

<p>
As amazing as the built-in CakePHP pagination helper is, sometimes you have to create your own custom queries for data you wish to paginate.  For <a href='http://www.ibl.org'>the simulation baseball league site</a> I wanted to be create an admin area where I could paginate through the series instead of individual games.  It took some digging around with google, but I found some info in a thread (thanks to <a href='http://www.3hndesigns.com/'>Baz</a> for contributing that tidbit to the thread) on how to do just that.
</p>

<p>
Okay, so it turns out that the paginate() method that is used to generate the data that you (oddly enough) paginate through takes the same arguments as Model::findAll().  So, if you want to use your own query, you simply create a 'paginate' method for your model.  In my case, I needed a query that would group things together (because a 'series' is simply a collection of all games between two teams in a particular week).  The query stuff I did is ugly because there currently is no support 'group by' in CakePHP (although that would be a very interesting project to tackle).  So, here's what I did:
</p>

<pre><code>/**
 * Custom paginate method
 */
function paginate($conditions, $fields, $order, $limit, $page = 1, $recursive = null) {
    $conditions[] ="1 = 1 GROUP BY week, away_team_id, home_team_id";
    $recursive = -1;
    $fields = array('week', 'away_team_id', 'home_team_id');

    $params = array(
          'conditions' =&gt; $conditions,
          'recursive' =&gt; $recursive,
          'fields' =&gt; $fields,
          'order' =&gt; $order,
          'limit' =&gt; $limit,
          'page' =&gt; $page
     );

     return $this-&gt;find('all', $params);
}
</code></pre>

<p>
So, that's the first part of the custom query stuff taken care off.  Next, I needed to be able to properly count the number of 'series' so that the numbers (you know, that stuff that says 'page 7 of 23') come out properly.  Again, you can override the paginateCount() method if you need to.  It uses the same parameters as Model::findCount().  Now, the custom query I'm using below is Postgres-specific, so YMMV:
</p>

<pre><code>/**
 * Custom paginateCount method
 */
function paginateCount($conditions = null, $recursive = 0) {
    $sql = "SELECT DISTINCT ON(week, home_team_id, away_team_id) week, home_team_id, away_team_id FROM games";
    $this-&gt;recursive = $recursive;
    $results = $this-&gt;query($sql);

    return count($results);
}
</code></pre>

<p>
So there you have it.  Hope this helps out anyone who's been trying to figure out how to use custom queries with their pagination.
</p>
