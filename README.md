const WaitEngine = require('../core/waitEngine');

class LoginActions {
  static async perform(step, envConfig) {
    const role = String(step.Role || '').trim().toLowerCase();
    const loginType = String(step['Login Type'] || '').trim().toLowerCase();

    if (!['sso', 'ias'].includes(loginType)) {
      throw new Error(`Unsupported login type: ${loginType}`);
    }

    const creds = envConfig.credentials?.[role];
    if (!creds) {
      throw new Error(`No credentials configured for role "${role}"`);
    }

    const username = creds.username || process.env.TEST_USERNAME;
    const password = creds.password || process.env.TEST_PASSWORD;

    if (!username || !password) {
      throw new Error(`Missing username/password for role "${role}"`);
    }

    // Step 1: enter username / email
    const usernameEl = await $(
      '#username, input[type="email"], input[name="loginfmt"], input[name="identifier"], input[type="text"]'
    );
    await usernameEl.waitForDisplayed({ timeout: 20000 });
    await usernameEl.setValue(username);

    // Step 2: click Next
    const nextBtn = await $(
      '//button[normalize-space()="Next"] | //input[@value="Next"] | //*[@role="button" and normalize-space()="Next"]'
    );
    await nextBtn.waitForDisplayed({ timeout: 20000 });
    await nextBtn.click();

    await WaitEngine.waitForStableUI();

    // Step 3: enter password
    const passwordEl = await $(
      '#password, input[type="password"], input[name="passwd"], input[name="password"]'
    );
    await passwordEl.waitForDisplayed({ timeout: 20000 });
    await passwordEl.setValue(password);

    // Step 4: click Sign in
    const signInBtn = await $(
      '//button[normalize-space()="Sign in"] | //input[@value="Sign in"] | //*[@role="button" and normalize-space()="Sign in"]'
    );
    await signInBtn.waitForDisplayed({ timeout: 20000 });
    await signInBtn.click();

    await WaitEngine.waitForStableUI();

    // Step 5: wait for your app landing page
    // Replace this with a real post-login element from your application
    const landingEl = await $(
      '//*[normalize-space(text())="Create Grievance"] | //*[@data-qa-type="tile"]//*[contains(normalize-space(.),"Create Grievance")]'
    );
    await landingEl.waitForDisplayed({ timeout: 30000 });
  }
}

module.exports = LoginActions;
