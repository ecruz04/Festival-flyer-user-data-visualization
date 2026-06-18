from flask import Flask, render_template, redirect, session, request, url_for, send_file
import spotipy
from spotipy.oauth2 import SpotifyOAuth
import os
from html2image import Html2Image

app = Flask(__name__)
app.secret_key = os.urandom(24)

CLIENT_ID = '41393dafd40e49feadd7eba283b81362'
CLIENT_SECRET = '1242925aa2b047089b8c463518356cd1'
REDIRECT_URI = 'http://127.0.0.1:5000/callback'
SCOPE = 'user-top-read'

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/login')
def login():
    sp_oauth = SpotifyOAuth(
        client_id=CLIENT_ID,
        client_secret=CLIENT_SECRET,
        redirect_uri=REDIRECT_URI,
        scope=SCOPE,
        cache_path=None  
    )
    auth_url = sp_oauth.get_authorize_url()
    return redirect(auth_url)


@app.route('/callback')
def callback():
    sp_oauth = SpotifyOAuth(
        client_id=CLIENT_ID,
        client_secret=CLIENT_SECRET,
        redirect_uri=REDIRECT_URI,
        scope=SCOPE,
        cache_path=None
    )

    session.clear() 

    code = request.args.get('code')
    token_info = sp_oauth.get_access_token(code, as_dict=True)
    session['token_info'] = token_info

    return redirect(url_for('flyer'))


def get_spotify_client():
    token_info = session.get('token_info', {})
    if not token_info:
        return redirect(url_for('login'))
    return spotipy.Spotify(auth=token_info['access_token'])

@app.route('/flyer')
def flyer():
    sp = get_spotify_client()
    mode = request.args.get('mode', 'artists')
    time_range = request.args.get('time_range', 'medium_term')
    
    if mode == 'songs':
        top_tracks = sp.current_user_top_tracks(limit=20, time_range=time_range)['items']
        entries = [{
            'label': f"{t['name']} – {t['artists'][0]['name']}",
            'url': t['external_urls']['spotify']
        } for t in top_tracks]
    else:
        top_artists = sp.current_user_top_artists(limit=20, time_range=time_range)['items']
        entries = [{
            'label': a['name'],
            'url': a['external_urls']['spotify']
        } for a in top_artists]

    headliners = entries[:3]
    mid = entries[3:10]
    small = entries[10:]

    return render_template('flyer.html',
                           headliners=headliners,
                           mid=mid,
                           small=small,
                           mode=mode,
                           time_range=time_range)

@app.route('/download')
def download_flyer():
    mode = request.args.get('mode', 'artists')
    flyer_url = f"http://localhost:5000/flyer?mode={mode}"
    hti = Html2Image()
    hti.output_path = './'
    hti.screenshot(url=flyer_url, save_as='flyer.png')
    return send_file('flyer.png', as_attachment=True)

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('thank_you'))

@app.route('/thankyou')
def thank_you():
    return render_template('thankyou.html')

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=True, use_reloader=False)
