const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

exports.handler = async function(event) {
  if (event.httpMethod !== 'POST') {
    return { statusCode: 405, body: 'Method Not Allowed' };
  }

  try {
    const { artworkId, title, price, imageUrl } = JSON.parse(event.body);

    if (!title || !price) {
      return { statusCode: 400, body: JSON.stringify({ error: 'Missing required fields' }) };
    }

    const siteUrl = process.env.URL || 'https://henk-weijers-site.netlify.app';

    const sessionParams = {
      payment_method_types: ['card'],
      line_items: [{
        price_data: {
          currency: 'eur',
          product_data: {
            name: title,
            description: 'Original acrylic painting by Henk Weijers',
          },
          unit_amount: Math.round(parseFloat(price) * 100),
        },
        quantity: 1,
      }],
      mode: 'payment',
      success_url: siteUrl + '/payment-success.html?artwork=' + encodeURIComponent(title) + '&id=' + encodeURIComponent(artworkId || ''),
      cancel_url: siteUrl + '/works/view.html?id=' + encodeURIComponent(artworkId || ''),
      metadata: {
        artwork_id: artworkId || '',
        artwork_title: title,
      },
    };

    if (imageUrl) {
      sessionParams.line_items[0].price_data.product_data.images = [imageUrl];
    }

    const session = await stripe.checkout.sessions.create(sessionParams);

    return {
      statusCode: 200,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ url: session.url }),
    };
  } catch (err) {
    console.error('Stripe error:', err);
    return {
      statusCode: 500,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ error: err.message }),
    };
  }
};
